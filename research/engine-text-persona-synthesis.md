# Text + Persona Synthesis — engine pivot research

> Compiled: 2026-09-05 · Scope: keep the pipeline, swap the voice-conversion engine for a **text + persona-reference + delivery-guide** controllable TTS engine (line/span-level control of emotion, pacing, prosody). Constraint: fully local on 2×T4 (Kaggle, 30 GB total / 16 GB per card), no paid API, client-facing MIT license per ADR-0001.

## Candidate engines — summary table

| Model | Zero-shot clone | Control-at-any-point | License (code / weights) | Per-voice | VRAM on T4 | Persona consistency | Source |
|---|---|---|---|---|---|---|---|
| **CosyVoice 3.0** `Fun-CosyVoice3-0.5B-2512` | ✅ ~3–10 s ref, cross-lingual | ✅ instruction (emotion/speed) per prompt; inline markup `<strong> […]`, `[laughter]`, `[breath]` on Instruct variant | **Apache-2.0 / Apache-2.0** | ~0.5 B | 16 GB OK (~3–4 GB claimed, unverified) | strong (multi-turn, cross-lingual) | github.com/QwenAudio/CosyVoice |
| **Qwen3-TTS** 12Hz-1.7B / 0.6B | ✅ **Base** clone (3 s) | ⚠️ instruction works on **CustomVoice/VoiceDesign tracks only — NOT on the clone Base track** (GH #218, #231) | **Apache-2.0 / Apache-2.0** | 0.6–1.7 B | 16 GB OK (fp16) | strong on Base; instruction gap unresolved | github.com/QwenLM/Qwen3-TTS, arXiv 2601.15621 |
| **Orpheus** 3B (L3.2-3B base) | ⚠️ few-shot via pretrained prompts (no dedicated clone model) | ✅ true **inline per-span tags** (`emotions.txt`: `#calm# #pleading# …`; emotive `<laugh> <sigh> …`); ~200 ms streaming | **Apache-2.0 / Apache-2.0** (HF card) — **gated download (must accept form)** | 3 B (8-bit ~4 GB) | 16 GB OK | English-only; identity from prompt pairs | github.com/canopyai/Orpheus-TTS; HF `canopylabs/orpheus-3b-0.1-ft` |
| **Zonos v0.1** | ✅ few-second ref or speaker embedding | ✅ **per-utterance dials**: speaking rate, pitch variation, audio quality, 8-emotion vector; 44 kHz | **Apache-2.0 / Apache-2.0** | ~1.6 B | 16 GB OK | strong | github.com/Zyphra/Zonos |
| **ZONOS2** (Jun 2026) | ✅ high-fidelity clone | ✅ speaking-rate 8 speeds, emotion vectors + strength, realtime | ⚠️ repo says **MIT**, Zyphra site says **Apache-2.0** (flag) | MoE 8 B total / 0.9 B active | 16 GB tight; made for 1×H100-class | strong | github.com/Zyphra/ZONOS2 |
| **IndexTTS-2.5** (Aug 2026) | ✅ single ref clip | ✅ **precise duration mode** + `duration_factor` 0.5–2.0×; 8-emotion vectors, Qwen text-emotion, **inline tags**; 5 langs | ⚠️ **custom bilibili Model Use License** (not clean Apache) | ~0.8 B | 16 GB OK | strong (timbre/emotion decoupled) | github.com/index-tts/index-tts; HF `IndexTeam/IndexTTS-2.5` |
| **GPT-SoVITS v4** | ✅ ~5 s ref / 1 min few-shot | ⚠️ ref-audio + global speed dial only; **no inline per-span emotion** | **MIT / MIT** | ~1–2 B | 16 GB OK | good | github.com/RVC-Boss/GPT-SoVITS/releases (v4 2025-04, v2Pro 2025-06; no v5) |
| **Chatterbox** (Resemble) | ✅ zero-shot | ✅ emotion **exaggeration slider**; paralinguistic tags in Turbo | **MIT / MIT** | 0.35–0.5 B | <8 GB | mid | github.com/Resemble-AI/Chatterbox |
| **XTTS-v2** | ✅ 6 s ref, 17 langs | ❌ no per-span control | MPL-2.0 / CPML | 0.7 B | 16 GB OK | good | github.com/coqui-ai/TTS |
| **F5-TTS** | ✅ 3–15 s ref, CLI speed dial | ❌ no per-span; speed via prompt text length | MIT code / **weights CC-BY-NC-4.0 (Emilia)** → **excluded** | 0.34 B | ~3 GB | good | github.com/SWivid/F5-TTS (+ issue 709) |
| **Spark-TTS** | ✅ 0.5 B Qwen2.5, zh/en | ❌ tangential (voice *creation* dials only) | Apache code / **weights CC-BY-NC-SA-4.0** → **excluded** | 0.5 B | small | mid | github.com/SparkAudio/Spark-TTS; HF card (License Update) |
| **Fish-Speech 1.4/1.5, S2, S2 Pro** | ✅ | ✅ inline emotion tags (S2) | **CC-BY-NC-SA / Fish Audio Research License – commercial needs paid license** → **excluded** | 0.3–4 B | fits | strong | github.com/fishaudio/fish-speech LICENSE; HF s2-pro; fish.audio/s2 |
| **Seed-TTS family** | ✅ (ICL/2.0) | ✅ (best-in-class instructor) | **weights NOT released — API-only** → **excluded** | n/a | n/a | n/a | github.com/BytedanceSpeech/seed-tts-eval; Seed-TTS 2.0 (Oct 2025) |
| **MegaTTS3** | ⚠️ cloning needs **upload to ByteDance → latents back** (not self-contained) | ⚠️ accent-intensity only; duration "coming soon" | Apache-2.0 / Apache-2.0 | 0.45 B | small | n/a (gated path) | github.com/bytedance/MegaTTS3 |
| **NVIDIA MagpieTTS / Magpie Zeroshot** | ⚠️ Zeroshot NIM is **gated (approval form)** + **NVIDIA Open Model License**; base Magpie explicitly "not intended for zero-shot cloning" | ❌ preset 16 emotion voices only | Apache-2.0 framework / **NVIDIA Open Model License** | 0.36 B | NIM container | n/a | github.com/NVIDIA-NeMo/Speech; HF `nvidia/magpie_tts_multilingual_357m`; docs.nvidia.com NIM TTS |
| **DiTTo-TTS / PromptTTS 2** | research | research | research-only | — | — | — | ICLR 2025; HF paper 2309.02285 |

## Verified control capability detail (per model, cited)

- **CosyVoice 3.0 (FunAudioLLM/QwenAudio)** — Repo Apache-2.0, ~23k★. `Fun-CosyVoice3-0.5B-2512` open-sourced Dec 2025: 0.5 B, zero-shot cloning from a short reference, **instruction control** (emotion/speed) as part of the prompt, pronunciation-inpaint, 9 languages + cross-lingual clone, streaming. The older `CosyVoice-300M-Instruct` line supports **inline markup**: `<strong>…</strong>`, `[laughter]`, `[breath]`, plus a voice-conversion function. Cross-lingual cloning is its standout (secondary 2026 comparison ×1 confirms). VRAM "~3–4 GB" is a secondary claim — **unverified**.
- **Qwen3-TTS (QwenLM, Apache-2.0, released 2026-01-21)** — Lineup: `12Hz-1.7B/0.6B-Base` (3-second voice cloning), `1.7B-CustomVoice` (9 preset premium timbres + **instruction control** of emotion/prosody), `1.7B-VoiceDesign` (build a voice from a description + instruction control). 10 languages, streaming (~97 ms claim). **Critical gap:** the *clone* track does NOT do instruction control — GH discussions #218 (add conductor to cloning) and #231 (instruction param on Base does nothing). The "Conductor" model from the tech report is absent from the released table → **treat as not released**. Long-speech ">10 min seamless" is a tech-report/card claim — **unverified**.
- **Orpheus (canopyai, 3B, Apache-2.0)** — FS `canopylabs/orpheus-3b-0.1-ft` HF card: `license: apache-2.0`, **gated** ("agree to share your contact information"). Llama-3.2-3B base; 24 kHz SNAC; ~200 ms streaming (Baseten fp8/fp16). **Real per-span inline tags** at any text position (`#calm# #pleading# …` + emotive `<laugh> <chuckle> <sigh> …`). Zero-shot clone is **few-shot prompting** on the pretrained model (no dedicated clone model) — more text-speech prompt pairs → better identity. English only. fp16 shards ≈15 GB (a 26 GB `optimizer.bin` rides along — don't download it).
- **Zonos (Zyphra) v0.1** — Apache-2.0, 1.6 B, 200k+ training hours, **44 kHz native**, clone from a few-second reference (or speaker embedding). Conditioning gives **fine-grained dials**: speaking rate, pitch variation, audio quality, emotion (happiness/fear/sadness/anger). Controls are **per utterance**, not inline — so per-span control is achieved by synthesizing per sentence with per-sentence conditioning. **ZONOS2** (2026-06-12): MoE 8 B/0.9 B active, realtime, emotion strength + 8-speeds rate control — but 16 GB VRAM on a T4 is tight and the repo/LICENSE-vs-site license wording conflicts (**MIT vs Apache-2.0** — verify before use).
- **IndexTTS-2 / 2.5 (Bilibili IndexTeam)** — Autoregressive zero-shot clone (GPT + flow-matching + BigVGAN, ~0.8 B, 22.05 kHz). Unique: **precise duration control** (mode 1 = specify token count for exact duration; mode 2 = free AR preserving prompt prosody) + **`duration_factor` 0.5–2.0×** speaking-speed control (2.5) + emotion decoupled from timbre (8-emotion vectors, Qwen-based text emotion, per-segment inline tags). En/zh (2.0); +ja/es/ar (2.5). ⚠️ **License:** HF `IndexTTS-2.5` card = `bilibili-model-license` (custom restrictive agreement; index-tts2.org marketing claims "Apache 2.0 / free for commercial" — **conflict, needs counsel**. As of the 2025-09 release note, *duration-control was "not yet enabled in this release"* — check before relying on the exact-token mode.
- **GPT-SoVITS v4** — MIT. Release history tops out at **v4 (2025-04-22)** and **v2Pro (2025-06-06)**; **no v5** exists through 2026-09. Ref audio sets speed/emotion in zero-shot + a global speed dial; **no inline per-span emotion tags**. Few-shot fine-tune from 1 min for a locked persona.
- **Excluded for license/access:** Seed-TTS/2.0/Audio (API-only, official "will NOT be releasing weights"); Fish 1.4/1.5 (CC-BY-NC-SA) & S2/S2 Pro (Fish Audio Research License — any commercial purpose incl. internal ops needs a paid license); F5-TTS weights & Spark-TTS weights (CC-BY-NC); NVIDIA Magpie/Magpie-Zeroshot (NVIDIA Open Model License, gated approval, base variant explicitly not clone-capable); MegaTTS3 (Apache code but **voice latents must be obtained by uploading a sample to ByteDance** → not self-contained/local).

## Gate-impacts (`spec/acceptance-gates.md`)

| Gate (as written) | Verdict under the pivot | Redefined to |
|---|---|---|
| DNSMOS P.835 OVRL ≥ 3.6 / SIG ≥ 3.8 | **Survives** (same metric, measured on TTS output) | unchanged — resynthesis must clear the bar; TTS usually cleaner than VC noisewise |
| UTMOS ≥ 3.8 | **Survives** | unchanged |
| Pacing ±5 % target (±8 % hard) vs **donor** | **Dies** — TTS recomposes timing; no engine re-speaks the donor's rehearsal pacing | **Pacing-guide adherence**: each line's measured duration within ±5 % (hard ±8 %) of the *intended seconds* set in the delivery guide. `duration_factor` (IndexTTS2/Zonos/GPT-SoVITS/CosyVoice speed) is the control surface |
| Length ratio 0.95–1.05 vs **donor** | **Dies** (same reason) | length ratio vs text-derived target from the script/guide (IndexTTS2 exact-token mode can hit this directly) |
| F0 mean ≤ ±1.5 st vs **donor** | **Dies** — contour is re-composed; per-frame f0 match is impossible in TTS | **Banded F0**: line median F0 within ±1.5 st of the *persona reference median* (intra-line dynamic variation allowed); drop per-syllable alignment |
| SECS vs reference clip | **Survives** (now vs persona reference) | unchanged; add **cross-utterance SECS drift** across a chapter (identity consistency) |
| Loudness −16±1.5 LUFS / TP ≤ −1.5 dBTP | **Survives** (post-processing, engine-agnostic) | unchanged |
| Blind A/B | **Survives / expanded** | A/B engines vs each other and vs persona reference on a fixed script |
| *(new)* Instruction-adherence | — | tag/guide → delivery hit-rate (did `#sad#` / "+angry" produce a rated-sad line?) |

## Compute & license feasibility on 2×T4

- **All shortlist engines fit one signed T4** fp16/int8: CosyVoice3 (0.5 B), Qwen3-TTS (0.6–1.7 B), Orpheus (3 B, use 8-bit), Zonos-v0.1 (1.6 B), IndexTTS-2.5 (0.8 B), GPT-SoVITS v4. **ZONOS2 (8 B MoE) is the only borderline case** — needs a 40 GB-class card or heavy quantization. Compute is therefore *not* the blocking constraint; license and control are.
- **License-compatible and self-contained (no API, no uploads, no gates):** CosyVoice 2/3 (Apache-2.0), Qwen3-TTS (Apache-2.0), Zonos v0.1 (Apache-2.0), GPT-SoVITS (MIT), Orpheus (Apache-2.0, gated-by-form but free), Chatterbox (MIT), XTTS-v2 (MPL/CPML). All are compatible with the MIT client pipeline per ADR-0001 (redistribute LICENSE + attribution notice).
- **Risk flags:** IndexTTS licenses via a custom *bilibili* agreement (get legal sign-off); ZONOS2 license wording conflict (MIT vs Apache-2.0); Orpheus = English-only + gated download; MegaTTS3 clone requires sending a reference to ByteDance (violates "local/no API"); Fish/F5/Spark/Magpie excluded outright.

## Verdict: **Switch** to text+persona synthesis

**Stay VC?** No — the client's asks (persona recreation from a reference + *per-span* delivery control of emotion/pacing) are exactly the text+persona TTS design; VC gives no per-span emotion control and cannot add guides, only transfer the existing performance.

**Recommended single engine: CosyVoice 3** (`Fun-CosyVoice3-0.5B-2512`) — the only shortlisted engine that is (a) Apache-2.0 code **and** weights, (b) **zero-shot clone + instruction control in one model**, (c) cross-lingual, (d) locally runnable on one T4, (e) streaming for long chapters. Its instruction path (emotion/speed per utterance) plus per-sentence synthesis maps cleanly onto the delivery-guide format. If per-span *inline tags* are non-negotiable at the text level, drop in **Orpheus** (Apache, true inline tags, English) or use **IndexTTS-2.5**'s inline emotion tags + duration control after legal sign-off.

**Watch list (likely 2026-27 wave):** Qwen3-TTS **Conductor** (clone + instruction in one model — the exact gap this pivot needs; not yet in the release table); **Seed-TTS Lite** open-sourcing; ZONOS2 license clarification; F5-TTS/Spark commercial-weight releases.

**Hybrid note:** keep the VC engines behind the ADR-0001 subprocess seam as a secondary path — VC is still the only tool that preserves a *specific* donor performance's timing/intonation, useful for any future gate that genuinely needs matching, and for post-synthesis polish layers.

## Sources (primary)

- CosyVoice (Apache-2.0, CosyVoice3-0.5B-2512, instruct markup, VC function): https://github.com/QwenAudio/CosyVoice
- Qwen3-TTS (Apache-2.0, release 2026-01-21, model lineup, GH #218/#231, tech report): https://github.com/QwenLM/Qwen3-TTS · https://huggingface.co/collections/Qwen/qwen3-tts · arxiv.org/abs/2601.15621
- Seed-TTS "no weights" statement: https://github.com/BytedanceSpeech/seed-tts-eval · Seed-TTS 2.0/Seed Audio 1.0 announcements
- GPT-SoVITS releases (v4 2025-04-22, v2Pro 2025-06-06, no v5): https://github.com/RVC-Boss/GPT-SoVITS/releases
- Orpheus (Apache-2.0 repo + HF card, gated, tags): https://github.com/canopyai/Orpheus-TTS · https://huggingface.co/canopylabs/orpheus-3b-0.1-ft
- Zonos v0.1 / ZONOS2 (license discrepancy): https://github.com/Zyphra/Zonos · https://github.com/Zyphra/ZONOS2
- Fish-Speech licenses (Research License, CC-BY-NC-SA cards): https://github.com/fishaudio/fish-speech/blob/main/LICENSE · https://huggingface.co/fishaudio/s2-pro · https://huggingface.co/fishaudio/fish-speech-1.5
- F5-TTS (MIT code, CC-BY-NC weights, issue 709): https://github.com/SWivid/F5-TTS · https://huggingface.co/SWivid/F5-TTS
- Spark-TTS (Apache code, CC-BY-NC-SA-4.0 weights card, License Update): https://github.com/SparkAudio/Spark-TTS · https://huggingface.co/SparkAudio/Spark-TTS-0.5B
- MegaTTS3 (Apache code; clone latents via upload, accent-intensity control): https://github.com/bytedance/MegaTTS3
- NVIDIA MagpieTTS (NeMo Apache framework; NVIDIA Open Model License; "not intended for zero-shot cloning"; gated Zeroshot NIM): https://github.com/NVIDIA-NeMo/Speech · https://huggingface.co/nvidia/magpie_tts_multilingual_357m · https://docs.nvidia.com/nim/speech/latest/tts/voice-cloning.html
- IndexTTS-2 / 2.5 (custom bilibili license, duration + emotion + inline tags, arXiv 2506.21619 / 2601.03888): https://github.com/index-tts/index-tts · https://huggingface.co/IndexTeam/IndexTTS-2.5
- Chatterbox (MIT, emotion slider/Turbo tags): https://github.com/Resemble-AI/Chatterbox · XTTS-v2: https://github.com/coqui-ai/TTS
- DiTTo-TTS: ICLR 2025 · PromptTTS 2: https://huggingface.co/papers/2309.02285
- OpenVoice V2 (MIT, base-TTS + tone-color module — reference architecture for "synthesis + style"): https://github.com/myshell-ai/OpenVoice
- Secondary 2026 hands-on comparison (used only to corroborate license/feature table): https://neosophie.com/en/blog/20260317-tts

*Verification flags carried in text: CosyVoice3 VRAM (~3–4 GB) = secondary claim; Qwen3 ">10 min long speech" = tech-report claim; IndexTTS license = custom (conflicting marketing), duration mode "not yet enabled" per 2025-09 release note; ZONOS2 license wording = MIT-vs-Apache conflict; NVIDIA Magpie Zeroshot = gated approval + NIM container (not plain weights).*
