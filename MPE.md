# Felucca MPE

By Stanley Gurvich, Emerald Moon Labs. MIDI Polyphonic Expression for the FM-1, on top of Felucca 1.6.1 (Leo Kuroshita,
Hügelton Instruments, GPL-3.0-only).

**Tested only in the emulator, not on a real FM-1.** Played there with a ROLI Seaboard RISE 49 and an Embodme Erae
Touch (Mk 1): it works and feels like any other MPE synth. Flashing, USB MIDI on the device and its CPU are untried.

## Using it

MENU > MIDI > MIDI IN: **MPE** (after TRACK). A lower zone plays the **selected track**:

- **Channel 1** is the master: its bend (±2 semitones, RPN 0 changes it), mod wheel, expression and sustain act
  on the whole track, as SEL plays them.
- **Channels 2..16** are the members: a note each. A member's pitch bend moves its own note only (ROLI Glide),
  ±48 semitones by default (the MPE default; RPN 0 on the channel sets it, up to 96).
- Two fingers on the same key are two voices, each bending on its own; letting go of one ends only its voice.
- A released note keeps the bend it had through its release; the channel is free for the next note at once.
- MONO / LEGATO: one voice that follows the latest finger (its channel and bend). UNISON: the finger's stack bends together.
- DRUM tracks ignore member bends. The ARP's notes follow no channel (they play after the fingers).
- The FM-1 sounds 8 voices in all: the 9th finger steals, as for any notes.
- Every other MIDI IN setting plays as upstream Felucca (the host tests' golden renders are unchanged), except that
  RPN 0 takes a bend range up to 96 semitones on any routing (upstream: 24), so a Seaboard's 48 works in CH1-4 too.
- **Mock MPE without MPE mode:** MIDI IN CH1-4 and the controller on channels 1..4 (or CH5-8 and 5..8): each finger
  plays its own track with its own bend. With the same sound on tracks 1..4 that is 4-finger MPE on any routing.
- MIDI IN starts as CH1-4 and is saved with the project: set MPE after loading the firmware.

ROLI: Seaboard / LUMI in MPE mode (ROLI Dashboard), lower zone, 15 member channels; the bend range 48 (the default).

### Press and Slide: PRES and TIMB (milestone 2)

The MOD matrix (MOD page, SRC) has two more sources after SLEW: **PRES** (Press: the finger's channel pressure) and
**TIMB** (Slide: its CC74). Each voice has its own, from its finger's channel, smoothed (about 3 ms), so every
finger modulates only its own note. Route them like any source:

- **PRES → AMP** (AMT up): a finger sounds as hard as it is pressed (swells, no sound at rest).
- **PRES → CUT**, **TIMB → CUT**: a filter per finger. **PRES → PITCH** (small AMT): vibrato by pressing.
- On a per-track destination (PAN, the sends, E1..E8, RATE, DEPTH) they take the latest finger's value.
- A member's pressure no longer moves the track's AT, nor its CC74 the track's CC74 map: in MPE they are per finger.
  The master channel's (channel 1) still are the track's.
- Outside MPE, PRES is the track's channel aftertouch (as AT) and TIMB is 0.

### VLFO (works on any MIDI IN)

A matrix source after TIMB: the track's LFO read at each voice slot's own phase (slot k at k × 45°), so the notes of
a chord move out of step instead of together. VLFO → PITCH (note) with AMT 1..2 and a slow RATE: drift / ensemble;
VLFO → CUT (note): a chord that shimmers note by note. It follows the LFO's RATE, SYNC, WAVE, FADE and the matrix's
DEPTH; it is always bipolar (POL UNI does not apply), and WAVE S&H gives every voice the same value. On a per-track
destination it is the plain LFO.

### DRIFT (works on any MIDI IN)

A matrix source after VLFO: a slow random wander of each voice's own, the LFO left free. Every voice heads for a new
random target every ~3 s and glides there, each at its own point of the cycle, so the notes of a chord drift apart
and back without repeating. DRIFT -> PITCH (note) with AMT 1 (about ±15 cents) or 2 (about ±30): analog drift;
DRIFT -> CUT (note): a filter that breathes per note. The rate is fixed (a knob would cost a parameter: see ideas/).

## Where it is

| File | What |
| --- | --- |
| `firmware/src/core.h`, `params.c` | `ROUT_MPE` 6: MIDI IN's seventh value, "MPE" (after upstream 1.5.5's TRACK) |
| `firmware/src/seq.c` | MPE listens on every channel (as SEL); entering / leaving MPE (`mpe_route`) |
| `firmware/src/midi_control.c` | member channels: their bend (`mpe_ch_bend`), range 48 / RPN 0 up to 96, the note's channel into its voice, note-offs by channel |
| `firmware/src/core.h` | the MPE state shared by mod.c, voice.c, midi_control.c (`mpe_*`) |
| `firmware/src/voice.c` | each voice's channel (`mpe_vch`) and own smoothed bend (`mpe_vbend`) on top of the track's; same-note voices per channel; its Press / Slide (`mpe_vpres`, `mpe_vtimb`) |
| `firmware/src/mod.c`, `params.c` | sources PRES and TIMB (`MS_PRES`, `MS_TIMB`), per voice |
| `tests/midi_control_test.c` | `mpe_test`, `mpe_same_note_test`, `mpe_switch_test`, `mpe_press_test`, `mpe_mono_test` |

State added: about 0.5 KB of RAM (per voice: channel, bend, press, slide; per channel: bend, press, slide). Projects are unchanged
except G_ROUTE 6 (MPE), which upstream Felucca does not know (1.6.1 clamps it to TRACK, 1.5.1 and earlier to their
last value): set MIDI IN back before moving a project to upstream firmware. Projects saved by the 1.5.1-MPE builds stored
MPE as 5, which is upstream 1.5.5's TRACK: they open as TRACK (channels 1..4 to tracks 1..4); set MPE again.

## Testing without the FM-1

- Setup: `git submodule update --init` (upstream's engines), then generate the headers (`./build.sh`, or
  `tools/build.py`'s `generate()`, see [BUILDING.md](BUILDING.md)).
- Host tests: Felucca's suite, `tests/run_tests.sh`, or one test: `cc -O1 -w -Ibuild/gen -Ifirmware/src -o
  build/t/mc tests/midi_control_test.c -lm && build/t/mc`. The MPE tests in `tests/midi_control_test.c`: `mpe_test`,
  `mpe_same_note_test`, `mpe_switch_test`, `mpe_press_test`, `mpe_mono_test`, `vlfo_test`, `drift_test`; the second
  core's in `tests/dualcore_test.c` (VLFO, DRIFT and PRES on its parts; `run_tests.sh TSAN=1` under ThreadSanitizer).
- The browser emulator: Felucca's own (`web/emu`, built with Emscripten, BUILDING.md); play an MPE controller into it
  over Web MIDI.

## Next

1. RPN 6 (MPE Configuration Message): zone size from the controller; upper zone.
2. Lift (release velocity) as a source.
