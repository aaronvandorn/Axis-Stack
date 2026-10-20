# Axis Stack

A Kaoss Pad–style XY performance pad for phone, tablet and desktop browsers, with MIDI in and out.

**Live:** https://aaronvandorn.github.io/Axis-Stack/ (once GitHub Pages is enabled)

## What it does

- **Pad:** a two-axis touch surface that plays a looping sample through effect chains driven by X and Y.
- **Stacked axes:** each axis drives any number of effects at once, each with its own range, curve and fixed settings. For example, Y bends pitch ±12 semitones while a low-pass filter on the same axis opens wider the further the pitch is bent.
- **Chains page:** build and reorder the effect chain for each axis. You can also move effects between axes and set which chain the audio passes through first.
- **Side controls:** a source switch (4 sample slots, tapes from Tape Loop Deck, or Discobot voices when hosted), 4 axis sets (A–D), a control loop (record, play and erase pad gestures, optionally snapped to the sample loop), Hold, and a touch-to-sound mode.
- **Effects:** pitch, filter (LP/HP/BP/notch), drive, bitcrush, delay, reverb, chop/gate, ring mod, flanger, pan, level.
- **Setup page:** load your own audio into any slot and set loop points, MIDI Learn for X/Y/touch and every button, and MIDI CC output of the pad position.
- Dark hardware-console theme by default, with a Light/Dark toggle.

## Tapes from Tape Loop Deck

Open [Tape Loop Deck](https://aaronvandorn.github.io/Tape-Loop-Deck/) in another tab of the same browser and set **Source → Deck** on the Pad page. Every tape on the deck shows up as a key; pick one and the pad plays a copy of it, locked to the deck, through the axis chains. The deck keeps playing its own tape, so the pad layers on top.

- The copy starts and stops with the deck, follows the tape's speed and any edits, and goes quiet while the deck mutes the tape (mute, solo or a pattern switch, timed to land on the same line).
- Pitch on the pad goes through the pitch shifter rather than the playback speed, so the copy never slips off the deck's time.
- **Play** sounds or silences the copy; **Touch** sounds it only while you touch the pad. Keys `1`–`4` pick the first four tapes.
- If the deck is reloaded, the pad finds the tape again by name.
- In a MIDI export the copy is a held note from 64 up, in the deck's tape order.

This uses the shared `deck-link.js` module (the `DECK-LINK` block in `index.html`). Same browser and same site only.

## Record & export

The **Record & export** group at the bottom of the Pad page saves WAV and/or MIDI. The MIDI file holds the pad's movement as two controller lanes (the X and Y CC numbers and channel set under MIDI, 12 and 13 by default), starting from wherever the pad sat when the take began, plus a held note for the sample that was sounding (60 for slot 1, 61 for slot 2, and so on). Takes are measured in seconds.

While the app's sound is on, the last 48 seconds of what it plays (audio and notes) are kept in memory. **Save last N** takes what just played; **Record next N** starts recording now. With a bar grid running it works in bars (1 to 16), cut on exact bar lines so takes loop seamlessly; otherwise in seconds (5 to 45). Tick **WAV** for exactly what you heard, **MIDI** for the notes, or both for one .zip with the two files. Nothing is uploaded. This comes from the shared `jam-clips.js` module (the `JAM-CLIPS` block in `index.html`), which every app in the set uses.

## Discobot guest

Axis Stack can sit in the rack of [Discobot](https://iw978599.github.io/discobot/), a browser sequencer and drum machine, as a guest instrument. In Discobot choose **Project → Add Guest Instrument** and paste `https://aaronvandorn.github.io/Axis-Stack/`. Discobot then:

- starts and stops the sample loop on its beat, and fits the loop to a whole number of its beats (tape-style: the speed follows the tempo);
- locks control loops recorded while it plays to its beat count;
- takes Axis Stack's sound into its mixer;
- saves Axis Stack's settings with each scene.

**Discobot voices as the source.** With a Discobot that shares its voices, set **Source → Discobot** on the Pad page and pick a synth lane, the drum bus or a single drum. The voice leaves Discobot's mix and plays through the pad's axis chains, with pitch shifting applied live. This needs the optional `input` extension of the guest protocol on Discobot's side. [DISCOBOT-PROMPT.md](DISCOBOT-PROMPT.md) is the prompt that adds it.

`discobot-test-host.html` is a small stand-in for Discobot's side, with test voices, a click, tempo changes and settings recall. Use it to try all of this without Discobot: serve the folder (`python -m http.server 8000`) and open `http://localhost:8000/discobot-test-host.html`.

Opened on its own, Axis Stack is unchanged.

## Keyboard

`Space` play · `1`–`4` sample · `Q` `W` `E` `R` axis set · `L` record loop · `P` play loop · `H` hold

## Default MIDI map

| Control | Default |
|---|---|
| Pad X / Y | CC 12 / CC 13 |
| Samples 1–4 | Notes 36–39 |
| Axis sets A–D | Notes 40–43 |
| Loop rec / play | Notes 44 / 45 |
| Hold / Play | Notes 46 / 47 |

Web MIDI works in Chrome, Edge and Firefox on desktop and Android. iOS browsers do not support it.

## Hosting

This is a single self-contained `index.html` with no build step. Enable GitHub Pages under Settings → Pages → Deploy from branch → `main` / root.
