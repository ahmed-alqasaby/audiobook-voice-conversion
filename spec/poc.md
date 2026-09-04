# One-Chapter Proof — compute-minimal POC plan

Goal: convert **one donor chapter** into **4 character voices** and pass **every gate** in
`spec/acceptance-gates.md` on **all 4 voices**, then export 4 reusable presets — on a
**2×T4 (30 GB) Kaggle** budget. If all gates pass → the showcase is publishable.

## Donor (verified)

- *The War of the Worlds* (v5), read by Cliff Stone — LibriVox, cataloged 2021, solo narrator.
- Chapter **"14 — In London"**, 24:20.
- 64 kbps: `https://archive.org/download/waroftheworlds_2103_librivox/waroftheworlds_14_wells_64kb.mp3`
- 128 kbps alt: `https://archive.org/download/waroftheworlds_2103_librivox/waroftheworlds_14_wells_128kb.mp3`
- Falls backup: Cori Samuel v3 (ch.16), Mark Nelson Time Machine (ch.4).

## Flow (compute-minimal ordering)

1. **Preprocess** — ffprobe the donor (sample rate / channels / noise headroom); resample to
   model rate; loudness-normalize; silence-strip; chunk into utterance-sized segments;
   write a chunk **manifest (JSON)** so every later stage is deterministic.
2. **Reference clips** — derive 4 per-voice 5–30 s references from donor segments via
   DSP (vocal-tract-length shift + EQ) per `spec/voice-bible.md`. Non-donor embeddings cost 0.
3. **Engine A — Seed-VC V2 zero-shot** (minutes/voice, no training):
   `length-adjust 1.0` · `f0-condition False` (prose → carry donor F0) · `diffusion-steps 30` · fp16.
   Order: Mara → Narrator → Kael → Motive (low-risk first).
4. **Fast-core gates** after each voice: DNSMOS P.835, UTMOS, pacing, length ratio.
   Pass ⇒ continue. Fail ⇒ one retune (reference/params), then escalate to Engine B.
5. **Engine B — RVC v2 fine-tune** (only on gate failure, one voice at a time):
   ~5–10 min clean target audio, feature extraction (HuBERT + RMVPE), train ~100–200 epochs
   (≈60 min on one T4), Faiss retrieval index; convert; re-run step 4.
   The engine that passes is the per-voice winner (ADR-0001 logic).
6. **Post** — adaptive EQ + de-ess + `loudnorm` to **−16 LUFS / TP −1.5**.
7. **Full gates** — f0 drift, SECS (measure donor/render set first → calibrate floor in
   `spec/acceptance-gates.md`), blind A/B panel.
8. **Export presets** — YAML per voice (`spec/../presets/<voice>.yml`): engine, reference,
   conversion params, post profile, and that voice's gate numbers (Q7 schema).
9. **Gate report** — every render's number vs every threshold. Publish iff 4/4 pass all gates.

## Compute budget

Seed-VC path ≈ **1–2 Kaggle GPU-hours total**. RVC escalation is charged only if a voice
actually fails (expected: Motive). This keeps the "not much compute" constraint on the table.

## Blocking facts still open

- Confirmed sample rate / noise floor of the 64 kb variant (ffprobe on download).
- SECS floor after calibration (step 7).
- Whether derived reference clips give sufficiently distinct timbres for 4 believable voices.