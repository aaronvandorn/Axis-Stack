# Axis Stack

A Kaoss Pad–style XY performance pad for phone, tablet and desktop browsers, with MIDI in and out.

**Live:** https://aaronvandorn.github.io/Axis-Stack/ (once GitHub Pages is enabled)

## What it does

- **Pad:** a two-axis touch surface that plays a looping sample through effect chains driven by X and Y.
- **Stacked axes:** each axis drives any number of effects at once, each with its own range, curve and fixed settings. For example, Y bends pitch ±12 semitones while a low-pass filter on the same axis opens wider the further the pitch is bent.
- **Chains page:** build and reorder the effect chain for each axis. You can also move effects between axes and set which chain the audio passes through first.
- **Side controls:** 4 sample slots, 4 axis sets (A–D), a control loop (record, play and erase pad gestures, optionally snapped to the sample loop), Hold, and a touch-to-sound mode.
- **Effects:** pitch, filter (LP/HP/BP/notch), drive, bitcrush, delay, reverb, chop/gate, ring mod, flanger, pan, level.
- **Setup page:** load your own audio into any slot and set loop points, MIDI Learn for X/Y/touch and every button, and MIDI CC output of the pad position.
- Dark hardware-console theme by default, with a Light/Dark toggle.

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
