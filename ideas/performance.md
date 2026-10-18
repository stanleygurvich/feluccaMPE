# Performance ideas

| Idea | What it rests on | Status |
| --- | --- | --- |
| **Faux MPE / alternating channels**: e.g. channels 1..3 alternating over three tracks with the same sound, one clean channel for drums | MIDI IN CH1-4 already plays a channel per track with its own bend (measured, stock Felucca); an editor preset could set it up | verified (routing), not built |
| **Per-finger pan** (Slide moves each note left / right) | pan is one value per track (`mix_part` / `mix_spread`, fx.c); SPREAD puts voices on two sides only | verified: needs per-voice stereo rendering: CPU per voice per sample, **not measured** |
| **Per-finger resonance** | RES is a knob read per track; only PITCH / CUT / SHP / AMP are per voice in the matrix | verified: needs a per-voice resonance input to ANALOG's filter |
| **MPE on drums / MPE-controlled FX** | DRUM tracks ignore member bends today (by design) | idea only |
| **RPN 6** (the controller sets the MPE zone; upper zone) | RPN parsing exists for RPN 0 (midi_control.c) | verified (the parser), not built |
| **Pressure -> volume per note** | PRES -> AMP (note) | done (works now) |
