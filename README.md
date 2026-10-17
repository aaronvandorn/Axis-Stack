# Axis Stack

A Kaoss Pad–style XY performance pad for phone, tablet and desktop browsers, with MIDI in and out.

**Live:** https://aaronvandorn.github.io/Axis-Stack/ (once GitHub Pages is enabled)

## What it does

- **Pad:** a two-axis touch surface that plays a looping sample through effect chains driven by X and Y.
- **Stacked axes:** each axis drives any number of effects at once, each with its own range, curve and fixed settings. For example, Y bends pitch ±12 semitones while a low-pass filter on the same axis opens wider the further the pitch is bent.
- **Chains page:** build and reorder the effect chain for each axis. You can also move effects between axes and set which chain the audio passes through first.
- **Side controls:** a source switch (4 sample slots, or Discobot voices when hosted), 4 axis sets (A–D), a control loop (record, play and erase pad gestures, optionally snapped to the sample loop), Hold, and a touch-to-sound mode.
- **Effects:** pitch, filter (LP/HP/BP/notch), drive, bitcrush, delay, reverb, chop/gate, ring mod, flanger, pan, level.
- **Setup page:** load your own audio into any slot and set loop points, MIDI Learn for X/Y/touch and every button, and MIDI CC output of the pad position.
- Dark hardware-console theme by default, with a Light/Dark toggle.

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
