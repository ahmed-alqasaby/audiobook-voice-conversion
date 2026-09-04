# Voice Conversion — Emotion & Prosody Retention Research
### For: Audiobook Voice Conversion project (one clean narration → 4 character voices)
### Compiled: 2026-09-04 · Sources: arXiv papers, official repos/HuggingFace, official docs

---

## Model-of-Choice Recommendation (TL;DR)

**Primary: Seed-VC V2 (single speaker, zero-shot at inference, optional few-shot fine-tune).**
**Fallback / secret weapon: RVC v2 (per-voice fine-tuned checkpoint + retrieval index).**

- **Seed-VC V2** (`hubert-bsqvae-small`, ASTRAL-Quantization content encoder) is the best published
  zero-shot converter for **prosody/emotion retention with minimal timbre leakage** — it converts a
  source utterance's *content + prosody + pacing* into a target timbre without re-speaking the line,
  which is exactly what "one narration → N voices, same pacing" requires. Zero-shot at ~67M (CFM) +
  90M (AR) params — trivial on one T4. First-party report (arXiv:2411.09943) shows Seed-VC beats
  OpenVoice and CosyVoice on speaker similarity *and* intelligibility (WER/CER) and outperforms RVCv2
  on content preservation in singing; it also publishes a `length-adjust` dial for pacing.
- **RVC v2** is the community-proven workhorse for *per-voice stable output* (train with 5–50 min of
  a character's clean audio, top‑1 retrieval suppresses source-timbre leakage). Its published
  characteristics: trains on a T4/Colab (~60 min for a small set), and is the strongest stable-timbre
  baseline. It *can* drift on large-dynamic-range / highly expressive source material, so use the
  `index_rate` + `f0method` PHI to reduce artifacts.
- Why not default to so-vits-svc/DDSP-SVC: so-vits-svc is archived and singing-oriented; DDSP-SVC's
  raw output is articulatory/“robotic” and needs a diffusion/ref-low enhancer, and its pitch control
  is designed for *singing* F0, not read-speech pacing. Both are weaker on prosody preservation for
  narration.

**Recommended pipeline for this project**
1. **Seed-VC V2 zero-shot** for the 2 fiction + 1 non-fiction + 1 motivational voices straight from
   the single clean reference narration → 4 timbres, shared pacing. Fast, no training data needed.
2. If a voice shows **timbre instability or a "metallic" artifact**, fall back to **RVC v2** with a
   small per-character fine-tune (5–10 min of the target-character reference recordings) + retrieval
   index, then convert the same narration through it (Seed-VC/RVC EVAL notes both competitive on
   DNSMOS).
3. **Post-process**: NSFW-free adaptive EQ + light de-esser + loudness normalize (‑16‑23 LUFS for
   audiobook), and gate with DNSMOS P.835 (SIG/BAK/OVRL) + UTMOS to gate for artifacts; verify
   *pacing* by comparing speaking rate (syllables/sec via forced alignment / phone count) across each
   converted voice vs. the source.

---

## 1. State-of-the-art zero-shot / few-shot VC models

### 1.1 Seed-VC (ByteDance architecture; open release by Plachtaa) — ⭐ primary

- **What it is**: Zero-shot voice *and* singing voice conversion using a **diffusion transformer
  (U-ViT)** + an external **timbre shifter** during training to reduce timbre leakage, with SSL
  (HuBERT/Whisper) content features. Paper: *Zero-shot Voice Conversion with Diffusion Transformers*
  — **arXiv:2411.09943**. Official open repo: **github.com/Plachtaa/seed-vc** (GPL‑3.0); HF:
  **huggingface.co/Plachta/Seed-VC**.
- **Emotion/prosody retention**: It converts the *source utterance* directly (speech-to-speech),
  so the source emotion, prosody, and pacing are carried over by frame-aligned length regulation
  rather than re-synthesized. This is the key property for this project: **the same narration keeps
  the same pacing across all four converted voices.** `--length-adjust` (default 1.0) gives explicit
  pacing control. F0 conditioning is available for singing (SVC) but for read prose you keep
  `f0-condition=False`, which preserves the source's natural F0 contour (prosody).
- **Artifact level**: Competitive DNSMOS; the paper's Tables show Seed-VC and RVCv2 within noise of
  each other on SIG/BAK/OVRL. Reported main limitation is a slight trade-off of SIG/OVRL for higher
  similarity — i.e., watch for tonal "harshness" in the high range, which the EQ/de-ess post-step
  handles.
- **Requirements / GPU**: v1 offline model 98M params (Whisper-small), real-time model 25M (XLSR-tiny),
  SVC 200M, **V2 67M (CFM) + 90M (AR)** — all run comfortably in **fp16 on a single T4 (16GB)**;
  2×T4 (30GB) is ample. `diffusion-steps` 25 default (30–50 for best quality, 4–10 for speed).
- **Fine-tuning vs zero-shot**: Primarily **zero-shot** (works with no training data — just a
  reference clip). Repo also ships **few-shot/one-shot fine-tuning** (`train.py`, presets including
  `config_dit_mel_seed_uvit_whisper_small_wavenet.yml` for offline VC), so you can lock a voice.

### 1.2 FreeVC / FreeVC-slim (free-vc.github.io)

- **What it is**: Text-free, one-shot VC built on **VITS** with a WavLM info-bottleneck for clean
  content extraction. Paper: **arXiv:2210.15418**; open repo
  **github.com/OlaWod/FreeVC** (and slim variant **FreeVC-slim**).
- **Emotion/prosody**: Analysis-based, frame-aligned → preserves source prosody/pacing reasonably,
  but its spectral content (mel) decode is older (HiFi-GAN) and produces more "smooth/robotic"
  coloration on large-dynamic-range targets than diffusion models. **Not what you want as primary.**
- **Requirements/GPU**: light; trains on a single GPU, ~1–2GB VRAM feasible. Needs **training** per
  speaker pair (parallel/non-parallel; WavLM-based), though there are pretrained models for
  one-shot use.
- **Verdict**: Historically important baseline; outclassed for this project by Seed-VC/RVC.

### 1.3 RVC — Retrieval-based Voice Conversion (RVC WebUI)

- **What it is**: VITS-based convolutional VC that reduces *source-timbre leakage* by replacing
  source features with a **top‑1 retrieval** from the target speaker's training set. Open repo:
  **github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI** (MIT where applicable) +
  **huggingface.co/lj1995/VoiceConversionWebUI** pretrained weights.
- **Emotion/prosody**: Converts frame-aligned so source pacing/rhythm survive. However, because it
  maps every frame through the *retrieval* dictionary, **very expressive / large-dynamic-range source
  material can sound flattened or "muffled/robotic"** if the target set is small or the `index_rate`
  is set too aggressively. Mitigations: keep ≥10 min clean uniform target audio, tune `index_rate`
  (0–1) and use RMVPE f0 to prevent "muted-sound" (the repo explicitly lists this fix).
- **Artifact level**: Among the best community-tested on stable timbre; occasionally adds a slight
  "band-pass"/metallic sheen on high-dynamic content. Same as Seed-VC, the DNSMOS deltas vs Seed-VC
  are small.
- **Requirements/GPU**: Pre-trained base (VCTK ~50h). **Training a new voice is easy and fast even
  on poor GPUs** (FAQ: 5–50 min data recommended; 10–50 min ideal; 5–10 min acceptable for a clean
  distinctive timbre; runs on Colab T4 in ~60 min for 200 epochs / 6–7 min audio). Real-time GUI
  latency ~170 ms (90–170 ms).
- **Fine-tuning vs zero-shot**: **Requires fine-tuning a small per-voice model** (that's its strength:
  crisper timbre than pure zero-shot). Not truly zero-shot.

### 1.4 DDSP-SVC (yxlllc/DDSP-SVC)

- **What it is**: Singing voice conversion (SVC) built on **Differentiable Digital Signal Processing**
  (DDSP harmonic+noise subtractive synthesis) + optional enhancer / shallow diffusion / reflow model.
  Open repo **github.com/yxlllc/DDSP-SVC** (MIT).
- **Emotion/prosody**: DDSP's *harmonic+noise* source-filter model is **pitch-based (F0-conditioned)
  and tuned for singing**, not read prose. Raw DDSP output is "not ideal" (repo's own words —
  articulatory/robotic), improved to "no less than so-vits-svc/RVC" only after an enhancer or shallow
  diffusion. For narration pacing, its F0 focus adds little.
- **Artifact level**: The additive-synthesis core can produce a **metallic/sawtooth aliasing
  artifact** on high registers — the exact "metallic resonance" the client wants to avoid (see
  DDSP-QbE++ paper arXiv:2604.09246, which addresses supersaw aliasing). Needs post-enhancement.
- **Requirements/GPU**: Very light (~1–2GB, "near RVC training speed"); trains fast.
- **Verdict**: Not recommended for this audiobook-narration use case.

### 1.5 so-vits-svc (svc-develop-team/so-vits-svc)

- **What it is**: SoftVC VITS Singing Voice Conversion (AGPL‑3.0), open repo
  **github.com/svc-develop-team/so-vits-svc** (now **archived**).
- **Emotion/prosody**: Feeds HuBERT/ContentVec soft features straight into VITS, so pitch/intonation
  are *conserved*, but it's **singing-oriented** (F0/register editing, speaker-mix for anime
  characters). For read narration it's usable but not purpose-built. Cross-speaker prosody control
  is not a design goal.
- **Requirements/GPU**: 44.1kHz; v4 reduced VRAM significantly; diffusion (optional) needs more.
  Training needs a per-speaker dataset (often ~10 min+).
- **Verdict**: Archived; superseded by RVC/GPT-SoVITS for this use case.

### 1.6 Notable 2023–2026 alternatives

- **CosyVoice / CosyVoice 2 / CosyVoice 3** (Alibaba): LLM + flow-matching multilingual **TTS**
  with speech in-context learning; arXiv **2407.05407 / 2412.10117 / 2505.17589**. Zero-shot VC
  possible, high prosody naturalness, but it's *text-to-speech* (re-speaks text) rather than pure
  **timbre-conserving** frame conversion → **pacing is re-generated, not preserved** from your
  narration. Great for TTS synthesis but the wrong tool if you must keep the source narration's
  exact pacing.
- **GPT-SoVITS** (RVC-Boss): Zero-shot TTS (5 s) / few-shot (1 min) voice cloning; open repo
  **github.com/RVC-Boss/GPT-SoVITS** (MIT). It has **no built-in emotion tag injection** (community
  notes) but v3/v4 claims "easier to generate speech with richer emotional expression" via a more
  stable GPT; it *re-speaks* the text → pacing is regenerated. Good if you want to *re-perform* the
  narration in a character voice; not for exact-pacing preservation.
- **OpenVoice** (MyShell, arXiv **2312.01479**): Instant voice cloning with granular voice-style
  (emotion) control via a separate style/emotion branch. It's a cloning/TTS stack; strong emotion
  tags, but again re-synthesis rather than frame-aligned conversion.
- **Takin-VC** (arXiv **2410.01350**): expressive zero-shot VC with adaptive hybrid content encoding;
  explicitly targets *expressive-preserving* conversion — a strong research competitor to Seed-VC but
  less mature/already-published tooling.
- **PFlow-VC** (arXiv **2502.05471**, ICASSP 2025): conditional flow-matching VC with discrete pitch
  tokens for *expressiveness/prosody transfer*; relevant for pitch-conditioned prosody control.
- **REF-VC** (arXiv **2508.04996**): diffusion-transformer VC with random-erasing + implicit
  alignment + Shortcut-Model acceleration (4 steps); improves on Seed-VC for **noisy** sources and
  is expressive — good to watch, but Seed-VC is more battle-tested to date.

---

## 2. Papers on emotional/expressive & prosody-preserving VC

### 2.1 Emotional voice conversion (EVC)

- **TRACE-EVC** — *Text-Guided Relative Affective Control for Zero-Shot Emotional Voice Conversion*,
  **arXiv:2607.03666** (2026). Instruction-guided *relative* emotion edits ("make it calmer") via a
  source-anchored rectified flow (Emo-Compass). Preserves speaker identity, content, quality.
- **PromptEVC** — *Controllable Emotional Voice Conversion with Natural Language Prompts*,
  **arXiv:2505.20678** (Interspeech 2025). Natural-language emotion prompts + **prosody modeling and
  control pipeline "that adjusts the rhythm based on linguistic content and emotional cues"** —
  directly relevant to pacing/emotion. Explicit prosody manipulation.
- **Maestro-EVC** — *Controllable Emotional Voice Conversion Guided by References and Explicit
  Prosody*, **arXiv:2508.06890** (ASRU 2025). Independent control of content/speaker/emotion +
  **explicit prosody modeling with prosody augmentation**; captures temporal emotional dynamics.
- **ClapFM-EVC** — *High-Fidelity and Flexible Emotional Voice Conversion with Dual Control from
  Natural Language and Speech*, **arXiv:2505.13805**. CLAP-based emotion prompts + adjustable
  emotion intensity.
- **Emotion-Aware Prefix** — arXiv **2603.09120** (2026). Prefix conditioning for explicit emotion
  control in a two-stage VC backbone; dataset doubles ECA baseline.
- **EmoReg** — arXiv **2412.20359**. Emotion-intensity regularization in diffusion-based EVC.

### 2.2 Prosody / expressiveness preservation in VC

- **REF-VC** (arXiv **2508.04996**) — The *key paper on the "robotic/flattened" artifact*: it
  explicitly states **"Traditional ASR-based methods ensure noise robustness but **suppress prosody
  richness**," while SSL-based methods improve expressiveness** — the core design tension this whole
  project sits on. It solves it with random-erasing on SSL features.
- **Takin-VC** (arXiv **2410.01350**) — expressive zero-shot VC preserving source expressiveness.
- **PFlow-VC** (arXiv **2502.05471**) — discrete pitch tokens + flow matching for **prosody/emotion
  transfer** in timbre conversion (ICASSP 2025).
- **ZSVC** (arXiv **2501.04416**) — zero-shot *style* VC with disentangled latent diffusion +
  adversarial training.
- **ProsoCodec** (arXiv **2606.21888**) — *prosody-oriented speech codec for VC*, separating content/
  speaker/prosody — a future-looking representation for prosody-preserving conversion.
- **Vibrato/Singing-style control**: VibE-SVC (arXiv **2505.20794**) and its follow-up (arXiv
  **2606.17126**) target vibrato/F0 high-frequency contours for expressiveness (singing-oriented but
  conceptually relevant to large-dynamic-range targets).

### 2.3 The "robotic / metallic artifact" problem for large-dynamic-range targets

- The artifact is driven by two mechanisms: (a) **prosody suppression** from content bottlenecking
  (ASR-based) — documented in REF-VC (arXiv 2508.04996); and (b) **source-filter/DDSP aliasing** —
  a sawtooth "metallic" sound, documented in the DDSP-SVC lineage, e.g. **DDSP-QbE++** (arXiv
  **2604.09246**), which specifically describes *"abrupt discontinuities (that) introduce aliasing
  artefacts that manifest perceptually"* — and the EL-speech adaptation paper (arXiv **2601.03892**)
  which notes "constant pitch, limited prosody, and mechanical noise" as degradation.
- Mitigation in practice (from the RVC/Seed-VC official docs): reduce `index_rate`/`inference-cfg-rate`
  toward the target set, use RMVPE f0, and apply post-EQ. The diffusion-based methods (Seed-VC,
  REF-VC, PFlow-VC) inherently produce smoother, less aliased output than pure source-filter vocoders.

---

## 3. Objective quality metrics for artifacts & consistency

### 3.1 Speech-quality / artifact metrics

| Metric | Type | What it scores | Scale | Notes / source |
|---|---|---|---|---|
| **MOS** (subjective) | human | overall/listening quality | 1–5 | gold standard |
| **DNSMOS P.835** | no-reference NN | SIG, BAK, OVRL (signal, background, overall) | ~1–5 | Microsoft, arXiv **2010.15258** (base) + **2110.01763** (P.835); used in Seed-VC paper |
| **UTMOS** | no-reference NN | naturalness (nMOS) | ~1–5 | IEEE SLT 2022, arXiv **2204.02152**; repo tarepan/SpeechMOS; SST 4.4+ = near-natural |
| **NISQA** | no-reference NN | overall + noisiness/coloration/discontinuity/loudness | 1–5 | diagnostic — good for "why" an artifact appears |
| **PESQ** (ITU‑T P.862) | full-reference | codec/telephony quality | −0.5–4.5 | needs clean reference; weak for broadband/VC |
| **ViSQOL** | full-reference | MOS-LQO (1–5), spectro-temporal NSIM | 1–5 | google/visqol; 16k speech / 48k audio |
| **PESQ/POLQA** | full-reference | 3.5+ = high | — | needs aligned reference |

- **Recommended "clean" thresholds (practical, from community + published baselines):**
  - **DNSMOS OVRL ≥ 3.6–3.8** (and SIG ≥ 3.8) → "non-artifact" for converted speech; converted VC
    typically lands 3.2–4.0 depending on model/step count. Use this gate before shipping a voice.
  - **UTMOS (nMOS) ≈ 4.2–4.4** = studio-quality TTS; **≥ 3.8–4.0** = clean-ish; below ~3.5 ⇒ audible
    artifacts worth fixing.
  - **PESQ ≥ 3.5** and **ViSQOL ≥ 3.6** where a reference exists (e.g., resynth baselines).
  - These are **guidelines**, not standards with official "pass/fail" cutoffs — report the numbers
    plus a small blind-listening checkpoint.

### 3.2 Speaker similarity & intelligibility

- **SECS / cos-sim** (speaker embedding cosine similarity, e.g., WavLM/COSMOS) — Seed-VC reports SECS
  vs OpenVoice/CosyVoice. Higher = more target-like.
- **WER / CER** (ASR word & char error rate) — intelligibility/content preservation; Seed-VC uses
  SenseVoice for CER (arXiv 2411.09943). Lower = cleaner content transfer.

### 3.3 Prosody / pacing-consistency measurement (what the literature actually uses)

- **F0 RMSE / F0 Pearson correlation (PCC)** — how well converted pitch tracks the source/target.
  Standard in VC papers (F0RMSE, PCC sources cited in the ScienceDirect VC-eval review).
- **Speaking rate** = #syllables (from **forced alignment**, e.g., Montreal Forced Aligner /
  Whisper timestamps / phone counts) per second — direct measure of **pacing consistency**. Compare
  each converted voice's speaking rate to the source narration's; accept if within a small margin
  (e.g., <±5–8%).
- **Length ratio / duration delta** — Seed-VC's `length-adjust` and frame-aligned length regulation
  make duration-preservation measurable as output-length/source-length ≈ 1.0.
- **MCD / MSD** (mel/spectral cepstral distortion) — global spectral accuracy (older metric; noted
  limitation: doesn't capture prosody).
- **Jitter / Shimmer / HNR / VTL** — speech-production parameters correlated with perceived VC
  quality (ScienceDirect review): good for detecting "robotic" vs natural vocal perturbation.
- **Recommendation for this project**: gate each voice on DNSMOS P.835 (SIG/BAK/OVRL) + UTMOS, verify
  SECS against the intended character reference, and verify **pacing** via forced-alignment speaking
  rate on a shared test sentence across all 4 voices.

---

## 4. Practical feasibility on 2 × T4 GPUs (30 GB total)

| Model | Mode | Training data / length | Training time (T4/Colab) | VRAM for inf/train | Fit to 2×T4 |
|---|---|---|---|---|---|
| **Seed-VC V2** | **zero-shot** (optional few-shot) | none for zero-shot; fine-tune ~minutes each if locked | fine-tune short | fp16, fits one T4 | ✅ Excellent |
| **Seed-VC v1 offline** | zero-shot | none | — | 98M params, one T4 | ✅ Excellent |
| **RVC v2** | few-shot *per voice* | **5–50 min** clean (10–50 ideal) | ~60 min for small set (200 ep, 6–7 min audio) | low (works on 1060/3GB slow; comfortably T4) | ✅ Excellent |
| **GPT-SoVITS** | few-shot TTS (re-speak) | 5 s zero-shot / 1 min few-shot | 5–30 min (1 min data, RTX 4090) | ~6–12 GB | ✅ Fits, but re-speaks (pacing regenerated) |
| **CosyVoice 3** | zero-shot TTS (re-speak) | reference clip | — | larger; Multi-GPU helpful | ⚠️ Big; fits on 30GB but re-speaks |
| **DDSP-SVC** | per-voice fine-tune | 10 min+ | very fast (near-RVC) | 1–2 GB | ✅ But add enhancer/diffusion |
| **so-vits-svc** | per-voice fine-tune | ~10 min+ | moderate | low-moderate (v4 lighter) | ✅ But archived, singing-focused |

### Practical plan to get one stable character voice

1. **Clean, uniform dataset** is the #1 lever (RVC FAQ: 10–50 min low-noise recommended; 5–10 min
   works for clean/distinctive timbre; 1–2 min unrepeatable). Same quality bar applies to Seed-VC
   few-shot fine-tune.
2. **Seed-VC V2 zero-shot path** (fastest): pick a reference clip of the target character, convert
   the whole narration; verify DNSMOS/UTMOS + pacing. Fine-tune only if timbre instability appears.
3. **RVC fallback**: preprocess (resample to 40k/48k, slice, denoise), run feature extraction
   (HuBERT + RMVPE), train ~100–200 epochs on one T4 (< ~1 hr for small set), build the Faiss
   retrieval index, then convert. Tune `index_rate` and f0 method to control timbre-vs-artifact.
4. **Batch across 2 T4s**: train 2 voices in parallel (one GPU each) or pipeline train+convert;
   30GB is far more than any single model here needs, so you can also fit the diffusion model at
   higher precision / more diffusion steps for best quality.
5. **Post**: gate each voice with DNSMOS P.835 + UTMOS + speaking-rate check; EQ/de-ess/normalize to
   audiobook standards (44.1kHz WAV or 192 kbps MP3, ~‑16‑23 LUFS).

---

## Sources (primary)

1. Seed-VC paper — arXiv **2411.09943** (github.com/Plachtaa/seed-vc; HF Plachta/Seed-VC).
2. FreeVC paper — arXiv **2210.15418** (github.com/OlaWod/FreeVC).
3. RVC repo/docs — github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI (FAQ: data/training);
   HF lj1995/VoiceConversionWebUI.
4. DDSP-SVC repo — github.com/yxlllc/DDSP-SVC.
5. so-vits-svc repo — github.com/svc-develop-team/so-vits-svc.
6. GPT-SoVITS repo — github.com/RVC-Boss/GPT-SoVITS.
7. CosyVoice / 2 / 3 — arXiv **2407.05407 / 2412.10117 / 2505.17589**.
8. OpenVoice — arXiv **2312.01479**.
9. REF-VC — arXiv **2508.04996** (prosody-suppression + robotics).
10. Takin-VC — arXiv **2410.01350**; PFlow-VC — arXiv **2502.05471**.
11. TRACE-EVC — arXiv **2607.03666**; PromptEVC — arXiv **2505.20678**;
    Maestro-EVC — arXiv **2508.06890**; ClapFM-EVC — arXiv **2505.13805**;
    Emotion-Aware Prefix — arXiv **2603.09120**; EmoReg — arXiv **2412.20359**.
12. DNSMOS — arXiv **2010.15258** (+ P.835 **2110.01763**, microsoft/DNS-Challenge); UTMOS — arXiv
    **2204.02152** (tarepan/SpeechMOS); ViSQOL — google/visqol.
13. DDSP-QbE++ (metallic/aliasing artifact) — arXiv **2604.09246**; EL-speech prosody —
    arXiv **2601.03892**.
14. ProsoCodec (prosody-oriented codec) — arXiv **2606.21888**.
15. VC speech-production-parameter evaluation (jitter/shimmer/HNR/VTL vs MOS) — ScienceDirect
    *Computer Speech & Language*, 2025.
