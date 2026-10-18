# "Free" modulation sources

The principle: whatever the firmware already computes every block costs nothing to compute again. A new source
pays only for a lookup or a little arithmetic, and only while a MOD slot takes it (the fork's per-voice sources run
in `mpe_voice`, outside the audio ISR's loops, so the device budget holds).

## Done

| Source | What it reads | Measured device cost |
| --- | --- | --- |
| **PRES**, **TIMB** | a finger's channel pressure / CC74 (MPE) | (with MPE's bend) flash +224 B, RAM +512 B |
| **VLFO** | the track LFO at each voice slot's phase (slot k at k × 45°): a chord moves out of step | flash +76 B, RAM 0, ISR unchanged |
| **DRIFT** | a slow random wander per voice: one global block count, one seed, a hash per voice (~3 s per target, golden-ratio offsets so no two voices turn at once); the LFO stays free | flash +180 B, RAM +16 B, ISR ~+1 % |

## Candidates

| Idea | What it rests on | Status | Cost (estimate) |
| --- | --- | --- | --- |
| **BEND as a source** | each voice's own MPE bend (`mpe_vbend`) already kept: Glide amount -> CUT (note) brightens a bent note | verified (the value exists) | a lookup |
| **SLOT / SPREAD** | the voice slot number (0..7): a fixed fan of values across a chord (a deterministic RAND) | verified | a shift |
| **GLIDE** | `pitch_cur` vs `pitch16`: how far a note still is from its target (a filter that opens while it slides) | verified (both fields in `voice_t`) | a subtraction |
| **Opposite by SPREAD side** | `v->side` (SPREAD's left / right): a source that goes opposite ways left and right (stereo width from any source) | verified (field exists) | a negate |
| **LFO of another track** | each track's LFO is ticked anyway: track 2's LFO modulating track 1 | **unverified**: whether a silent track's LFO still ticks (track_render per part) | a lookup |
| **VLFO by note** | VLFO's phase offset from the note number instead of the slot (higher notes further along) | verified (VLFO's code) | a multiply |
| **LFO multiplier** | faster: the LFO phase shifted left (n × the rate, no state); slower: its own counter per track (16 B RAM) | verified (phase maths) | free / +16 B |
| **A drift / LFO-rate knob** | a real user parameter | verified: costs a parameter id, which changes P_COUNT: projects, presets, upstream compatibility, editors | expensive (formats), not CPU |
| **Sequencer position** | the step / bar position as a source ("where in the bar") | **unverified**: not read yet | ? |
| **Sustain pedal** as a source | the channel's pedal state (`midi_channel_t.pedal`) | verified (field exists) | a lookup |
| **Lift** (release velocity) | MIDI note-off velocity | verified: **not parsed today**: real new work | small |

## Custom-shape LFO (16 points, connected with a slew)

Asked 2026-10-13. Nothing built; costs are estimates from the code and docs read so far.

- **Running it is cheap:** per track and block, two of the 16 points picked by the LFO phase's top 4 bits and blended
  by the slew (stepped at none, a straight line at full): two table reads, a multiply, a shift, no divide; about the
  sine LFO's work, per track not per voice. VLFO follows it (it reads the LFO's wave).
- **A. Preset shapes:** e.g. 16 factory 16-step shapes appended to LFO WAVE (append-only enum: P_COUNT unchanged, no
  format change, saved with sounds / projects as a WAVE value). Flash ~300..400 B for 16 shapes (estimate), RAM 0.
  Slew fixed per shape (some stepped, some smooth). Recommended first.
- **B. User-drawn shapes:** 16 points + slew = 17 stored values. Upstream's EDITOR_PROTOCOL.md (1.5) says FUN10 has
  16 bytes to spare, room for 4 more parameters: 17 do not fit, so projects / user presets change format and part
  ways with upstream Felucca. Also needs an editor: pages of knobs on the FM-1, and / or drawing in a computer editor.
- **Middle:** user shapes kept in RAM only (lost at power-off), sent live from an editor: cheap in firmware, still needs
  an editor.
