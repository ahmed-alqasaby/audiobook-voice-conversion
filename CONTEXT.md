# Audiobook Voice Conversion

A pipeline that converts one clean reference narration into several distinct
character voices by timbre conversion (not re-synthesis), gated by numeric
acceptance thresholds, with reusable presets so a client can re-run it per book.

## Language

**Voice bible**:
A written spec of each target character voice — register, pacing, accent,
dynamic range — agreed before any tooling. It is the reference every pipeline
decision and every acceptance check is tested against.
_Avoid_: character list, voice spec (use the canonical "voice bible")

**Voice conversion (VC)**:
Speech-to-SPEECH timbre transformation: an existing recording's content,
prosody, and pacing are carried into a target voice. Distinct from TTS, which
re-speaks text and therefore regenerates pacing. This project is VC, not TTS.
_Avoid_: text-to-speech, voice synthesis (for this project's core path)

**Preset**:
A per-voice, machine-checkable config (model ref, reference clip, conversion
params, post profile, and the measured gate numbers) that lets a client re-run
a whole book without re-calibration.
_Avoid_: settings, config (when referring to the shipped, reusable artifact)

**Chapter**:
The unit of delivery and proof. One chapter is run end-to-end and must pass the
acceptance gates before the pipeline is considered valid for a full book.
_Avoid_: sample, demo clip (when it means the full chapter proof)

**Artifact**:
An audible defect in converted audio — "robotic"/metallic resonance, prosody
flattening, timbre drift. Quantified via DNSMOS/UTMOS/NISQA and the voice-D
emotion ratios, not by ear alone.

**Acceptance gate**:
A pre-committed numeric pass/fail threshold (clarity, loudness, pacing, f0,
emotion retention) fixed before audio is generated so acceptance cannot be
litigated after. See docs/tickets/03-acceptance-gates.md.
