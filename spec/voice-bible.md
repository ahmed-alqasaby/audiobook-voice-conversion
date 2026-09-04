# Voice Bible — the four showcase voices

The written spec of each target voice, agreed before any tooling. Every pipeline
decision and every acceptance check is tested against it (see `CONTEXT.md` → **voice bible**).

Composition per project brief: **2 fiction, 1 non-fiction, 1 motivational/emotional**.

## How a voice is defined

Each voice card specifies **register · pacing · accent · dynamic range**. These are the
agreed properties the reference clip must approximate and the gates must preserve.

> Showcase reality: we have **one donor** (the LibriVox chapter). The per-character
> **reference clips** (5–30 s timbre samples) therefore come from DSP-transforms of the
> donor itself — pitch/formant (vocal-tract-length) shift + EQ character — so the demo is
> self-contained and licensing-clean. The bible is the source of truth; the reference clips
> approximate it.

## Voice cards

### Mara — Fiction 1 (low register)
- **Register**: low, chest-dominant
- **Pacing**: deliberate, slightly slower than donor
- **Accent**: neutral (donor's native)
- **Dynamic range**: narrow–moderate
- **Artifact risk**: low (safe voice; validate first)

### Kael — Fiction 2 (bright/young)
- **Register**: higher, brighter timbre
- **Pacing**: similar to donor, lighter attack
- **Accent**: neutral → slight contemporary coloration
- **Dynamic range**: moderate
- **Artifact risk**: medium (bright/high-range harshness → EQ/de-ess watch)

### Narrator — Non-fiction (neutral authority)
- **Register**: mid, even
- **Pacing**: measured, steady
- **Accent**: neutral, RP-adjacent acceptable
- **Dynamic range**: narrow (flatness risk lives here — monotone is the failure mode)
- **Artifact risk**: low, but prosody-flattening is the trap

### Motive — Motivational/emotional (wide range)
- **Register**: versatile, mid–high lifts
- **Pacing**: energetic, expressive shifts
- **Accent**: neutral, emotive
- **Dynamic range**: WIDE — the artifact-risk center
- **Artifact risk**: **highest** — large dynamic range is where metallic/resonance and
  prosody-flattening appear (see `research/competitive-read.md`): validate last,
  expect the RVC fine-tune path here if gates fail.

## Fill order (compute-first)

1. Mara (low risk) and Narrator (flat-trap) first — cheap, shape the pipeline.
2. Kael — bright timbre tuning.
3. Motive last — wide-range voice; expect escalation to RVC fine-tune if Seed-VC flattens it.