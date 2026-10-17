# Prompt: let Discobot hand its voices to a guest (for Axis Stack)

This is a prompt for Ian, who maintains [Discobot](https://github.com/iw978599/discobot), to give
to Claude in the Discobot repository. It adds one optional feature, `input`, to guest protocol
version 1. With it, a guest instrument can take one of Discobot's own voices, such as a synth lane or
a drum, and play it back through its own processing.

Axis Stack already implements version 1 (transport, settings and sound), so it should work with
the Discobot that is live now. Only the voice hand-over needs this change. The Axis Stack side of it
is built and tested against a stand-in host (`discobot-test-host.html` in this repo), not yet
against Discobot itself.

Copy everything below the line.

---

I want Discobot to be able to **hand one of its voices to a guest instrument**. The first guest
that needs this is **Axis Stack** (https://aaronvandorn.github.io/Axis-Stack/, source at
https://github.com/aaronvandorn/Axis-Stack). It is a Kaoss Pad–style XY effects pad. Each axis
drives a stack of effects (pitch, filter, drive, bitcrush, delay, reverb, gate, ring mod, flanger,
pan). Today it plays its own sample loops. With this change the user can pick, inside Axis Stack,
"Synth 2" or "Snare". Discobot then takes that voice out of its own mix, streams it into the guest,
and gets it back processed through the guest's ordinary audio return.

Axis Stack is already a working version 1 guest (transport, state, audio). Please add this as an
**optional, additive feature of guest protocol version 1**, named `input`. Guests that do not list
`input` in `ready.features` must see no change at all. Nothing about a project without a handed-over
voice may change either.

## Read these first

- `docs/GUEST_PROTOCOL.md`: the protocol this extends.
- `ui/src/rack/GuestModule.tsx`, `ui/src/hooks/guestAudio.ts`, `ui/src/services/guests.ts`: the
  existing host side.
- `createAudioLane` and the lane and group routing in `ui/src/hooks/browserAudio.ts`, and
  `ui/src/services/mixerChannels.ts`: how a voice reaches the mixer.
- `ui/src/services/noteScheduling.ts` and `ui/src/services/drumScheduling.ts`: where notes and hits
  are timed. A handed-over voice has to be scheduled early (see Timing).
- Axis Stack's guest code, the `Discobot guest` section of its `index.html`, which is the other end
  of every message below.

## Messages

Same rules as the rest of version 1. Every message carries `discobotGuest: 1` and a `type`. Accept
messages only from the guest's own frame and origin, check every field, and ignore unknown types.

**Host to guest**

| `type` | Fields | When |
|---|---|---|
| `sources` | `sources`: array of `{ id, name, kind }` | After `hello`, but only if the guest listed `'input'` in its features. Again whenever the list changes: a lane added or removed, its model changed, a drum row added or removed. |
| `inputSelected` | `source`: id or `null` | In answer to every `selectInput`. Also unprompted whenever the hand-over ends for any reason (the voice was deleted, another guest took it, guest audio was turned off). |
| `input` | `source`, `wall`, `sampleRate`, `left`, `right` (`Float32Array`, same length, at most 16,384 frames, transferred) | Continuously while a voice is handed over, silence included. |

**Guest to host**

| `type` | Fields | Meaning |
|---|---|---|
| `selectInput` | `source`: an id from the latest `sources`, or `null` | Hand this voice over, or give it back. Anything not in the current list is treated as `null`. |
| `inputLate` | `ms` (number) | `input` blocks are arriving too late to play on time, by about this much. |

**Source ids** must stay the same across reloads and devices, because the guest saves its choice in
its own `state`, per scene:

- `synth:<lane id>` for each synth lane. The name is what the mixer shows, for example
  `Synth 2 · Juno`. `kind: 'synth'`.
- `drums` for the whole kit bus. `kind: 'drums'`.
- `drum:<instrument>` for each drum row, using the `DRUM_ROWS` ids (`drum:kick`, `drum:snare`,
  `drum:closedHH`, …). The name is what the drum grid shows. `kind: 'drum'`.

Do not offer other guests or the master bus. A voice goes to one guest at a time. If another guest
asks for a voice that is already handed over, answer that guest with `inputSelected: null` and
leave the first hand-over alone.

## Routing

While a voice is handed over it is an **insert**:

- Take the voice out of Discobot's mix completely: its direct path, its effect sends and its group
  return. What comes back through the guest's own channel (with the guest's volume and sends)
  replaces it. Otherwise the voice is heard twice.
- Tap it after its own level, pan, mute and solo, so muting the lane in Discobot still silences it,
  and before its sends.
- Give the voice back to the mix **at once** when the guest sends `selectInput: null`, when the guest
  is removed or reloads, when its `audio` is turned off, or when the voice is deleted. A 10 ms
  crossfade on each switch avoids a click. Settings come back per scene, so a song can switch a
  hand-over on or off at a bar line during playback.
- Mark it in the UI. The voice's mixer strip and its rack unit should say where it has gone, for
  example "→ Axis Stack", so the user knows why it is not in the mix.

## Timing

This has to land on the beat, the same way guest audio already does.

- The guest plays everything `latencyMs` early, and that now includes the `input` it receives. It
  places each block at its own `ctxAtWall(wall - latencyMs)`. Its output comes back stamped as usual,
  and the host holds it back by `latencyMs` as it already does.
- So **`wall` on an `input` block is when that audio would have been heard in Discobot's mix**, and
  the host has to produce it early enough for the guest to have it before it is due. Keep a per-guest
  `inputLeadMs`. Start it at `latencyMs + 120`, so 240 by default. Render the handed-over voice that
  much early: schedule its sequenced notes, drum hits and automation `inputLeadMs` ahead of the rest
  of the song, into its own lane that goes to the tap and not to the mix. Stamp each block with
  `wall = wallAtCtx(firstFrame / sampleRate) + inputLeadMs`.
- When the guest sends `inputLate`, raise `inputLeadMs` by `ms + 20` (up to about 1,000). Use the new
  value from the next note scheduled. When `latencyMs` for that guest goes up, raise `inputLeadMs`
  by the same amount.
- Notes played live (computer keyboard, MIDI in, keyboard audition) cannot be early. Send them as soon
  as they are played. They come back late by roughly the lead, which is fine for live playing. Say so
  in the help text for handed-over voices.
- Tempo changes and per-scene time signatures need nothing new: the early lane follows the same tempo
  map, shifted by `inputLeadMs`.

For the tap, the same 128-to-1,024-frame `AudioWorkletProcessor` the guest example uses works well.
Send stereo at the context's own rate; the guest resamples if it has to.

## Export

WAV export already plays guests through in real time and records their `audio`. A voice that is
handed over must be fed to its guest during that pass, rendered `inputLeadMs` early as above, and
left out of the offline render. If that is too much for now, export the voice dry (not handed over)
and say so in the export dialog. Do not drop it silently.

## Documentation

Add an "Input (optional)" section to `docs/GUEST_PROTOCOL.md` describing the above. In the "What the
host does not do yet" list, the line "The host does not send notes to a guest" stays true: this
sends audio, not notes.

## Leave alone

- Every guest that does not list `input`, and every project with nothing handed over.
- The existing `latencyMs` handling, `hello`, `transport`, `state` and `audio` messages.
- Project and scene storage, apart from what the guest already saves in its own `state`.

## Checking it

1. Open Discobot, choose **Project → Add Guest Instrument** and paste
   `https://aaronvandorn.github.io/Axis-Stack/`. The unit should read **Connected**. On Axis Stack's
   Setup page, the Discobot section should say how many voices are available.
2. Put a kick on every beat and press **Play All**. Axis Stack's sample loop starts on beat 1 and stays
   locked. This part uses only version 1 and needs no change to Discobot.
3. In Axis Stack, on the Pad page, set **Source → Discobot** and pick **Synth 1**. The synth leaves
   Discobot's mix and is heard through the pad. Moving the pad up bends its pitch and opens the filter.
4. Check timing. On Axis Stack's Chains page, switch every effect in the current set off so the voice
   passes through clean, then hand over **Kick** and compare it with Discobot's metronome. A flam (two
   close hits) means the early rendering or the stamp is off.
5. Change the tempo while playing, and change scenes in a song. Nothing should drop out or click.
6. Switch Axis Stack back to **Samples**, remove the guest, and turn its sound off. Each time the
   voice must come straight back into Discobot's mix.
7. Reload Discobot. The guest should ask for the same voice again and get it.
8. Open a guest that does not list `input` (the guest example) and check it behaves exactly as
   before.
