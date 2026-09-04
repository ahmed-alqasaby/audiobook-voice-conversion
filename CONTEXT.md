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

**Donor recording**:
The single clean, full-length narration that gets converted — the whole
chapter or book of audio. It is the source of the performance (pacing, prosody,
emotion) the pipeline must preserve. Contrast with a reference clip (below).
_Avoid_: source narration, input audio (ambiguous with reference clip)

**Reference clip**:
A short (5–30 s) per-character sample that defines a target voice's timbre for
conversion (the Seed-VC zero-shot reference, or the seed sample for an RVC
per-voice fine-tune). One per character voice in a preset.
_Avoid_: sample, voice clip, reference audio

**Character render**:
The per-chapter converted audio produced for one character voice. The artifact
the acceptance gates act on; one voice produces one render per chapter.
_Avoid_: converted voice, output clip, result audio

**Voice**:
The persistent character identity defined in the voice bible (register, pacing,
accent, dynamic range). Persists across chapters; a character render is one
production of a voice. Not the same as the timbre reference that approximates it.

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
litigated after. See research/vc-emotion-prosody.md §3 for the metric thresholds.
