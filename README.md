# Harmony

A browser-based chord synthesizer. Field Instrument 006 by M.B. Parks, Green Shoe Studio.

Press a key, hear a full chord. Designed to make rich, musical sounds accessible to people who don't read music or play an instrument, while still rewarding deeper musical exploration. Single-file HTML, runs entirely in the browser, no installation.

## Quick start

1. Open `harmony.html` in any modern browser (Chrome, Edge, Safari, Firefox).
2. Click "Tap to begin" on the splash screen to wake the audio engine.
3. Click a key on the on-screen keyboard, or press a letter key on your computer keyboard, to play a chord.

That's it. Everything else is optional refinement of the sound you're making.

## Playing

You play with two hands. The right hand picks notes, the left hand shapes the chord.

Right hand (notes):

| Key | Note  | Key | Note         |
|-----|-------|-----|--------------|
| A   | C     | W   | C# (black)   |
| S   | D     | E   | D# (black)   |
| D   | E     |     |              |
| F   | F     | T   | F# (black)   |
| G   | G     | Y   | G# (black)   |
| H   | A     | U   | A# (black)   |
| J   | B     |     |              |
| K   | C     |     |              |

Left hand (chord shaping, hover any button for a one-line mood description):

| Key | Type | Feel                  |
|-----|------|------------------------|
| Z   | Dim  | Tense, unsettled       |
| X   | Min  | Sad, melancholy        |
| C   | Maj  | Happy, bright          |
| V   | Sus  | Floating, unresolved   |
| B   | 6    | Soft, sweet flourish   |
| N   | m7   | Bluesy, dominant       |
| M   | M7   | Dreamy, jazzy          |
| ,   | 9    | Rich and full          |

## Controls

### Top row of dials

For each dial: drag vertically to change the value, scroll the mouse wheel for fine steps, click to push. Push usually opens a menu or toggles a mode. A small italic line below each label tells you in plain language what the dial does.

* **Sound** (Instrument). Cycle through synth presets.
* **Perform** (Play Style). Different ways the synth responds to your touch.
* **Key** (Mood). Push to cycle Off, Major, Minor. Turn to set the root note. The OLED header always shows the current key.
* **Bass** (Sound & Mode). Turn for bass instrument sounds. Push for the bass menu:
  * Off
  * Auto (bass follows your chord presses)
  * Doubled (bass doubles the chord an octave lower)
  * Hand-played (you play bass notes with the keyboard)
  * Bass Only (silences the chord voices)
* **BPM** (Tempo & Drums). Turn for tempo, push for the drum pattern menu.
* **Menu** (Settings). View options (Chord, Notes, Geek Out) and MIDI output devices.
* **Master** (Volume). Final output level after everything mixes together.

### Center stack (small dials)

* **Chord Shape**. How the chord is arranged from low to high. Different settings give the same chord a different feel, from tight and stacked to open and spread.
* **Bass Octave**. Shifts the bass note up or down by whole octaves.

### Channel strip (sliders)

Two stacked rows of compact sliders. All are directly editable: drag, scroll the mouse wheel, or focus and use arrow keys.

* **MIX row**: Synth, Bass, Backing. Independent volumes for the three audio sources before they reach the master.
* **FX row**: Reverb, Delay, Chorus, Drive. Effect intensities applied to the synth voices.

### Loop Console

A single state-aware primary button cycles through the recording states:

`RECORD → STOP → PLAY → PAUSE → RESUME`

The button text and color reflect the current state, so you always know what the next press will do. Other controls in the console:

* **Start Over**. Abandon the current take and reset the transport.
* **Save WAV**. Download the recording as 16-bit stereo audio. If you click while still recording, the recording is finalized first, then saved.
* **Load File**. Accepts any browser-decodable audio file (MP3, WAV, OGG, M4A, FLAC) and loops it as a backing track to play over. The filename appears next to the timer, with a small ✕ to clear it.

### Save Patch and Load Patch

Two brass buttons on the bottom of the chassis between the maker's plate and the Powered indicator.

* **Save Patch** writes a timestamped JSON file containing every user-controlled setting: sound preset, performance mode, key, transpose, bass settings, BPM, all four channel volumes, voicings, chord type selections, extensions, view mode, beats pattern, and FX intensities.
* **Load Patch** restores those settings from a file. Each field is validated independently, so a partially malformed patch still loads what it can.

Captured audio, the loop transport state, the MIDI device list, and the last-played chord are deliberately excluded from patches so the files stay portable.

## MIDI Out

If your browser supports Web MIDI and a hardware synth is connected, open the Menu and pick a MIDI output device. Notes are routed to three channels:

* Channel 1: chord notes
* Channel 2: bass notes
* Channel 10: drum hits (GM mapping)

Some embedded contexts (sandboxed iframes, certain previews) disable Web MIDI through Permissions-Policy. If the MIDI menu shows "Blocked by host page," save the file locally and open it standalone.

## Files Harmony writes

* `harmony-patch-YYYYMMDD-HHMMSS.json`. A complete settings snapshot.
* `harmony-loop-{timestamp}.wav`. A stereo 16-bit PCM recording from the Recorder.

## Browser requirements

Any modern browser with Web Audio API support. Tested on recent versions of Chrome, Edge, Safari, and Firefox. Audio requires a user gesture to start, which is what the splash screen handles.

## Architecture

Single-file HTML with all CSS, markup, and JavaScript embedded in one document. The script is divided into IIFE modules, each prefixed with a `=== module ===` header comment for easy navigation.

Module map:

```
EventBus     State        AudioCore    SynthVoice   SynthEngine
SoundBank    ChordEngine  KeyEngine    PerformanceEngine
EffectsChain BassEngine   LoopEngine   BeatsEngine
Patch        MidiOut      Display      UI           App
```

Modules communicate through `EventBus.emit(name, payload)` and `EventBus.on(name, handler)`. `State.set(partial)` merges into the central state object and emits a `state` event that any module can subscribe to. `AudioCore` owns the Web Audio graph and exposes named gain stages (master, synth, bass) so other modules don't reach into the audio context directly.

The audio routing graph is:

```
synth voices  -> synthGain  -> fxIn -> [reverb/delay/chorus/drive] -> fxOut --+
                                                                              |
                                                              dry path -------+--> master -> destination
                                                                              |
bass voices   -> bassGain   --------------------------------------------------+
backing audio -> backingGain -------------------------------------------------+
recorder tap  (ScriptProcessorNode, muted to destination, used only for WAV capture)
```

Extension points are marked with `// EXTEND:` comments throughout the source. To add a new synth voice, register it in `SoundBank` and add the corresponding voice class to `SynthEngine.VoiceTypes`. New drum patterns go in `BeatsEngine.PATTERNS`. New performance modes go in `PerformanceEngine.MODES`.

## Lineage

Harmony began as a software emulation of the Telepathic Instruments Orchid, then evolved into its own instrument with a different visual identity, additional features (Beats, MIDI Out, recorder with backing track, patch save and load, multi-channel mixer), and language aimed at making chord synthesis approachable to people without a music background.

## License

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

Copyright (C) 2026 M.B. Parks, Green Shoe Studio.

GPL-3.0
