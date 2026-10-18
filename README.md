# Felucca MPE

**MPE (MIDI Polyphonic Expression) for the M-Vave FM-1.** By Stanley Gurvich, Emerald Moon Labs.
Built on [Felucca](https://github.com/hugelton/Felucca) 1.6.1 by Leo Kuroshita, Hügelton Instruments (GPL-3.0-only).

> [!WARNING]
> **Tested only in the emulator, not on a real FM-1.** Everything so far ran off the device: played in the browser
> emulator (the same C source compiled to WebAssembly, so the synth engines and the MIDI handling are the real code)
> and checked by Felucca's host tests. What has
> **not** been tried is the FM-1 itself: flashing, USB MIDI on the device, and running on its CPU. The 1.6.1 build has
> also not been re-checked against the device's size and audio-timing budgets (the 1.5.1 numbers are below). If you
> flash it, you do so at your own risk: keep the official firmware at hand (M-VAVE's updater, or Felucca's web
> installer's **Return to official V15**).

## Played with real MPE controllers

Played in the emulator with a **ROLI Seaboard RISE 49** and an **Embodme Erae Touch (Mk 1)**: it works, and it feels
like any other MPE synth.

## What it does

Felucca turns the FM-1 into a deep little synth and groovebox. Felucca MPE lets you play it from an MPE controller,
with every finger its own voice and its own expression:

- **Each note bends on its own.** Slide one finger and only that note glides; the rest of the chord stays where it
  is. ±48 semitones by default (the MPE standard), up to ±96. Two fingers on the same key are two separate voices.
- **Press and Slide shape each note.** Two modulation sources, **PRES** (how hard you press) and **TIMB** (sliding
  along the key), go to any per-note destination: press harder for louder (AMP), slide to open the filter (CUT),
  press for vibrato (PITCH), or reshape the tone (SHP). Each finger moves only its own note.
- **Chords that breathe, on any keyboard.** Two more sources work without MPE too: **VLFO** runs the track's LFO at a
  different phase for each voice, so a chord's notes move out of step (ensemble, shimmer), and **DRIFT** gives each
  voice its own slow random wander, like an analog synth that never quite stays in tune.
- **A bigger bend range everywhere:** up to ±96 semitones on every MIDI IN setting (Felucca: ±24).
- **Everything else is Felucca 1.6.1, unchanged.** The same sounds, sequencer, effects and projects.

## Quick start

1. **Your controller:** MPE on, **lower zone**, **15 member channels** (channel 1 the master, 2 to 16 the notes),
   pitch-bend range **48**. That is the usual MPE default; on the Seaboard RISE 49 and the Erae Touch it is set in
   the controller's own editor software.
2. **The FM-1:** MENU > MIDI > MIDI IN: **MPE** (the last value, after TRACK). MPE plays the **selected track**.
3. **The track:** EDIT > VOICE: **POLY**, and an engine with plenty of voices (see below).
4. **Expression:** on the MOD page, set a slot's SRC to **PRES** or **TIMB** and its DST to AMP, CUT, SHP or PITCH;
   AMT sets how much. A good start: **PRES → AMP** (the note swells as you press) and **TIMB → CUT** (slide up to
   open the filter).

MIDI IN is saved with the project, so set it to MPE once per project. Full guide: **[MPE.md](MPE.md)**.

## Voices: how many fingers at once

The FM-1 has **8 voices**, shared by its four tracks. MPE plays one track (the selected one), so that track gets all
8 while the other tracks are silent. How many it can sound at once also depends on its **engine**, since each engine
has a voice cap of its own:

| Engine | Voices per track | Fingers in MPE |
| --- | --- | --- |
| ANALOG, PHASE, LOFI, SAMPLE, SLICE, TRIO, WHEEL, NOISE | 8 | up to 8 |
| **FM6** | **6** | up to 6 |
| VOICE | 4 | up to 4 |
| GRAIN, PHYS | 3 | up to 3 |
| DRUM | one per lane | notes play hits, finger bends ignored |

**FM6 is 6 voices:** FM6 is a six-operator FM engine, and each of its voices keeps a full six-operator state in the
track's engine memory; six of them fill it, so FM6 stops at 6. Play a seventh finger and one voice is taken over.

**Past the limit** (a 9th finger on ANALOG, a 7th on FM6, or fewer voices left because another track is sounding),
the new note takes a voice over: first the oldest released note (one ringing out), then the oldest held note, never
the lowest held note (your bass stays). The FM-1's own sequencer, playing other tracks at the same time, uses the
same 8 voices.

**Other voice modes:** MONO and LEGATO play one voice that follows your latest finger (its channel and its bend);
UNISON stacks voices on each finger and they bend together.

## The four new modulation sources

On the MOD page (SRC, after SLEW). On a per-note destination (PITCH, CUT, SHP, AMP) every voice gets its own value; on
the others (PAN, the sends, the engine knobs, RATE) they take the latest note's.

| Source | What it is | Try |
| --- | --- | --- |
| **PRES** | the finger's pressure (MPE channel pressure); outside MPE, the track's aftertouch | PRES → AMP: swells. PRES → PITCH, small AMT: vibrato by pressing |
| **TIMB** | the finger's slide along the key (MPE CC74); 0 outside MPE | TIMB → CUT: a filter per finger |
| **VLFO** | the track's LFO at a different phase for each voice | VLFO → PITCH, AMT 1–2, slow RATE: ensemble. VLFO → CUT: a shimmering chord |
| **DRIFT** | a slow random wander per voice (a new target every ~3 s) | DRIFT → PITCH, AMT 1 (±15 cents) or 2 (±30): analog drift |

VLFO and DRIFT work on any MIDI IN setting and on the sequencer's notes, not only in MPE.

## Good to know

- **Projects:** MIDI IN MPE is stored as value 6. Upstream Felucca does not know it (1.6.1 opens it as TRACK), so set
  MIDI IN back before moving a project to upstream firmware. Projects saved by the earlier 1.5.1-MPE builds stored MPE
  as 5, which is now Felucca's TRACK: they open as TRACK; set MPE again.
- **No MPE controller?** Set MIDI IN to CH1-4 and send each note on its own channel 1 to 4: each plays its own track
  with its own bend. With the same sound on tracks 1 to 4, that is 4-finger MPE.
- **Still to come:** the controller setting the zone itself (MPE's RPN 6), the upper zone, release velocity (Lift).

## Building and trying it

The same as Felucca: **[BUILDING.md](BUILDING.md)** for the firmware (`./build.sh`) and the browser emulator
(`web/emu`, with Emscripten). Upstream's engines are a git submodule: clone with `--recursive` or run
`git submodule update --init`. How the MPE code is laid out and tested: **[MPE.md](MPE.md)**.

## Credits and licence

Felucca MPE by **Stanley Gurvich, Emerald Moon Labs**. Felucca by Leo Kuroshita, Hügelton Instruments, with its
contributors (see [CREDITS.md](CREDITS.md)). GPL-3.0-only, as Felucca: see [LICENSE](LICENSE) and
[LICENSING.md](LICENSING.md). Not affiliated with M-VAVE, ROLI or Embodme.

## Changelog (one entry per increment)

| # | Version | What was added |
| --- | --- | --- |
| 1 | v1.5.1-MPE | **MIDI IN: MPE.** A lower zone on the selected track: channel 1 the master, channels 2..16 a finger each with its own **pitch bend (Glide)**, ±48 semitones by default. Two fingers on one key are two voices; a released note keeps its bend through its release. Host tests, and pitches measured in the WebAssembly build's audio. |
| 2 | v1.5.1-MPE | **Bend range up to 96 semitones on every MIDI IN** (upstream: 24), so a Seaboard's 48 also works in CH1-4 ("mock MPE": a finger per track). |
| 3 | v1.5.1-MPE | **Fix: no hanging note when MIDI IN changes while a note is held** (it hung until MIDI IN went back to CH1-4). Entering MPE drops the old routing's track bends. |
| 4 | v1.5.1-MPE | **Press and Slide per finger: the matrix's PRES and TIMB sources** (MOD SRC, after SLEW). Each finger's channel pressure and CC74, smoothed, on its own voice: PRES → AMP makes a finger sound as hard as it presses, TIMB → CUT a filter per finger. Outside MPE, PRES is the track's aftertouch. |
| 5 | v1.5.1-MPE | **Device build checked** (JieLi toolchain, upstream's `tools/build.py` checks and `tests/target_budget.py`): the per-voice MPE work left the audio interrupt's loops (shifts and a table instead of 4 divides; one helper called only while MIDI IN is MPE or a slot takes PRES / TIMB). Flash **+224 B** over upstream 1.5.1, RAM +512 B, audio-ISR static cost +5 % (within upstream's budget). |
| 6 | v1.5.1-MPE | **VLFO: a matrix source, the track LFO at each voice slot's own phase** (slot k at k × 45°). A chord's notes move out of step: ensemble, or drift at a slow RATE with a small AMT on PITCH (note) / CUT (note). Measured on a chord, PITCH AMT 6: LFO keeps the notes within 6 cents of each other, VLFO spreads them up to 158 cents. Always bipolar (POL UNI does not apply); WAVE S&H gives every voice the same value. Device: flash **+76 B**, RAM 0, audio ISR unchanged (it runs only while a slot takes it). |
| 7 | v1.5.1-MPE | **DRIFT: a slow random wander per voice, a matrix source**, the LFO left free. One global block count and one seed, hashed per voice (track × 8 + slot): a new random target every ~3 s, a straight glide, each voice at its own point of the cycle. Measured on a chord, PITCH (note) AMT 2, 12 s: each note ranges 34–58 cents on its own path. Device: flash **+180 B**, RAM +16 B, audio ISR ~+1 % (budget ok). Also `ideas/`: what else is nearly free, marked done / verified / unverified. |
| 8 | v1.6.1-MPE | **Felucca 1.6.1 underneath** (upstream merged in: 1.5.5 to 1.6.1, the PHASE / sample-pack licensing). **MIDI IN: MPE is now the 7th value (6)**: upstream 1.5.5 took 5 for TRACK, so projects saved with MPE by the 1.5.1-MPE builds open as TRACK; set MPE again. The MPE work follows upstream's second core (1.6.1, MENU > SYSTEM > 2ND CORE, beta): the voices' PRES / TIMB / VLFO / DRIFT are each part's own, so two cores rendering parts at once do not share them (upstream's dual-core test now carries these sources, clean under ThreadSanitizer). Fixed in the merge: MIDI IN TRACK listened on every channel. Host tests pass; the device build was not re-measured. |

### Device cost (measured, not estimated)

| | Upstream 1.5.1 | MPE fork | Room |
| --- | --- | --- | --- |
| App image (flash) | 495,588 B | 496,068 B (+480, with VLFO and DRIFT) | XIP region 581,564 B: ~84 KB free |
| RAM .data + .bss | 54,964 B | 55,492 B (+528) | of 73,728 B: ~18 KB free |
| Audio ISR (`fm1_alnk0_irq`), target_budget cost | 38,871 | 41,218 (budget 37,502, limit +10 %) | 15 divides in loops, as upstream |

Measured on 1.5.1-MPE by building the app with JieLi's toolchain (`tools/get_toolchain.sh`), without the SDK (packaging
only). Not yet re-measured on 1.6.1-MPE.

---

*Below: upstream Felucca's own README, unchanged.*


# Felucca

[![License: GPL-3.0-only](https://img.shields.io/badge/license-GPL--3.0--only-blue.svg)](LICENSE)

## Maintenance Notice

What began as a personal experiment grew far beyond my expectations. I’m grateful to everyone who contributed, supported the project, and made music with Felucca.

I’ve decided to step back from active development to focus on creating new instruments and music.

Felucca will remain available with occasional maintenance, but new features, issue responses, and pull request reviews are not guaranteed.

Thank you for everything. I hope you’ll look forward to my next creations.

## Overview

Multi-engine synthesizer firmware for the M-VAVE FM-1, version 1.6.1. Connect the FM-1 to a computer by USB, open the
[web installer](https://hugelton.github.io/Felucca/) in Chrome or Edge and press Install; no extra hardware is needed.
Installing is at your own risk: M-VAVE's updater or the installer's **Return to official V15** takes you back.
[Try it in your browser](https://hugelton.github.io/Felucca/webapp/try/), no FM-1 needed.

- Install: [web installer](https://hugelton.github.io/Felucca/)
- Edit: [web editor](https://hugelton.github.io/Felucca/webapp/editor/) ([Felucca-WebApp](https://github.com/hugelton/Felucca-WebApp))
- Read: the manual and the release notes are in the [wiki](https://github.com/hugelton/Felucca/wiki/Home)
- Build: [BUILDING.md](BUILDING.md)
- Related: [Felucca-Engines](https://github.com/hugelton/Felucca-Engines) (the ANALOG, LOFI, TRIO, WHEEL and NOISE engines, MIT; still maintained) ·
  [felucca-bsp](https://github.com/hugelton/felucca-bsp) (the FM-1 hardware layer)

## AI disclaimer

Felucca is developed with the assistance of AI coding agents (code, tests, documentation, translation, maintenance). The
design, features, sound design and direction are the maintainer's or the community's, and no generative AI is used for music,
icons or artwork. See [On AI-Assisted Development and Responsibility](https://github.com/hugelton/Felucca/discussions/166).

## Credits and licence

Free software: [GPL-3.0-only](LICENSE); the Hügelton Sample Pack is CC0 1.0; the engines in `firmware/engines` are MIT
([Felucca-Engines](https://github.com/hugelton/Felucca-Engines)); the bundled fonts and ported DSP keep their own licences
([LICENSES/](LICENSES/), [LICENSING.md](LICENSING.md)). Thanks to everyone whose work and contributions are in it: [CREDITS.md](CREDITS.md).

M-VAVE and FM-1 are trademarks of their respective owners. Felucca is not affiliated with or endorsed by them.

Copyright (C) 2026 Leo Kuroshita (@kurogedelic), Hügelton Instruments
