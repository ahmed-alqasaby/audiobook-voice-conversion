# Ecosystem Map — Real Models (HuggingFace) & Projects (GitHub) Closest to This Pipeline

> Compiled: 2026-09-04 · Method: 3 parallel research agents (Tier-1 WebSearch/WebFetch), each with one lane
> (HF models, GitHub projects, market position). Target: one clean narration → 4 preserved-prosody character
> voices, self-hosted 2×T4, gated (DNSMOS P.835 / UTMOS / pacing ±5-8% / f0 / blind A/B), reusable presets.

---

## 1. HuggingFace models — real repos, verified

| Model repo | Type | Likes | License | Params | Last update | Prosody-preserving? | Notes |
|---|---|---|---|---|---|---|---|
| [Plachta/Seed-VC](https://huggingface.co/Plachta/Seed-VC) | Audio-to-audio (zero-shot VC) | 92 | **gpl-3.0** | ~200M (v1); v2 hubert-bsqvae-small | no rev / model files in GitHub repo | ✅ frame-aligned, `--length-adjust`, `f0-condition` | 4 finetunes, **90 HF spaces** build on it — the S2S standard |
| [lj1995/VoiceConversionWebUI](https://huggingface.co/lj1995/VoiceConversionWebUI) | RVC weights (ONNX) | 1.21k | mit | ~1-2M/voice | no rev | ✅ timbre-only (preserves source prosody by construction) | **100 spaces** (Applio, AICoverGen, rvc-voice-cloner) |
| [IAHispano/Applio](https://huggingface.co/IAHispano/Applio) | RVC/VITS wrapper | 203 | mit | RVC backend | active | ✅ timbre-only | Turnkey RVC path on a single T4 — our fallback runner |
| [xihajun/GPT-SoVITS](https://huggingface.co/xihajun/GPT-SoVITS) | Few-shot VC + TTS | 7 | mit | ~1B | active | ⚠️ hybrid; VC mode keeps prosody | Backup when data ≤ 1 min/voice |
| [myshell-ai/OpenVoiceV2](https://huggingface.co/myshell-ai/OpenVoiceV2) | TTS (tone-color clone) | 499 | mit | ~500M | Apr 2024 | ⚠️ re-speaks; adds emotion/style | Not VC — only for deliberate emotion variation |
| [fishaudio/fish-speech-1.4](https://huggingface.co/fishaudio/fish-speech-1.4) | TTS | 460 | **cc-by-nc-sa-4.0** | ~0.7B | 2024 | ⚠️ re-speaks | ❌ non-commercial license → excluded |
| [coqui/XTTS-v2](https://huggingface.co/coqui/XTTS-v2) | TTS | 3.77k | coqui-public-model-license | ~800M | 2023 | ⚠️ re-speaks | 6-sec clone; the *de facto* audiobook baseline (see spaces below) |
| [ResembleAI/chatterbox](https://huggingface.co/ResembleAI/chatterbox) | TTS | 1.78k | mit | 0.5B (V3) | 2025 | ⚠️ re-speaks | Conversational TTS, not VC |

**Key data caveat:** HF does **not** track downloads for Seed-VC or RVC weights (empty README-era model cards), so the real popularity signals are **space usage + likes + GitHub stars**.

**Audiobook multi-voice already productized on HF — but all TTS-from-text (re-speaks):**
- `jkorstad/AudioBook` — narrator + 8 characters, XTTS-based
- `jkorstad/ebook2audiobook`, `drewThomasson/ebook2audiobook`, `zermok/ebook2audiobook` — ebook→audiobook XTTS pipelines

**Most-directly-usable ranking for us:**
1. **Seed-VC V2** — only open zero-shot speech-to-speech converter with true frame-aligned conversion (pacing/prosody preserved), `length-adjust` + `f0-condition` + semi-tone-shift per character, and fine-tune support. One T4 is plenty.
2. **RVC v2 via Applio** — per-voice fine-tune + retrieval index, MIT, CPU/T4 friendly; our fallback.
3. **GPT-SoVITS** — VC mode as plan C.

---

## 2. GitHub projects — verified status (2025-2026 activity)

| Repo | Stars | Last push | Status | License | Package | Fits our use? |
|---|---|---|---|---|---|---|
| [Plachtaa/seed-vc](https://github.com/Plachtaa/seed-vc) | ~3.9k | — | **⚠️ ARCHIVED (upstream frozen)** — confirmed via API | gpl-3.0 | Gradio UI + Python API + config presets + `train.py` fine-tune | ✅ Engine, but needs a **fork** to maintain |
| [RVC-Project/Retrieval-based-Voice-Conversion-WebUI](https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI) | ~38k | 2026-08 | ✅ Active | mit | CLI/Docker + `api_240604.py` | ✅ Our MIT fallback |
| [IAHispano/Applio](https://github.com/IAHispano/Applio) | active | 2026 | ✅ Active | mit | Turnkey RVC runner | ✅ Practical per-voice trainer |
| [RVC-Boss/GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS) | large | 2026 | ✅ Active | mit | WebUI + API | ⚠️ TTS-leaning; VC mode |
| [yxlllc/DDSP-SVC](https://github.com/yxlllc/DDSP-SVC) | — | — | maintenance | mit | CLI | ❌ singing/aliasing; rejected in research |
| [svc-develop-team/so-vits-svc](https://github.com/svc-develop-team/so-vits-svc) | — | — | **archived** | agpl-3.0 | CLI | ❌ |
| [0xfe/voice_forms?] noise-filtered — core ecosystem consolidated to Seed-VC / RVC / Applio / GPT-SoVITS | | | | | | |

**Best base repo (evidence-backed):** **fork `Plachtaa/seed-vc`** — the only OSS engine with true speech-to-speech zero-shot timbre conversion, explicit `--length-adjust` pacing dial, `--f0-condition`, YAML config presets, and a one-shot T4 fine-tune script. It is frozen upstream, so the fork IS the maintenance plan. **RVC WebUI (MIT, active, 38k★)** is the maintained fallback but with per-voice training + retrieval instead of zero-shot pacing preservation.

**The gates + post chain (DNSMOS P.835 / UTMOS / pacing / f0 / EQ / de-ess / LUFS) exist in NO base repo — that is our build scope either way.**

---

## 3. Market position — commercial & OSS multi-voice audiobook

| Product | Speech-to-speech (converts your audio)? | Multi-character | Pricing | Self-host? | Notes |
|---|---|---|---|---|---|
| ElevenLabs (Audiobooks / STS v2) | Marginal — STS exists but flagship flow is TTS-from-text | Yes (full-cast) | Credits: $6–$990/mo; Pro ≈ one 10-hr book; revisions re-bill | No (cloud lock-in) | Official v3 expressiveness-vs-consistency tradeoff reddit-thrashed; no objective gates |
| Play.ht / PlayAI | Partial (voice-changer + dialogue endpoints) | Yes | ~$31/mo + API credits | No | Vendor-migration risk already documented |
| Speechify | No (either TTS or translation dubbing) | Weak audiobook flow | 3-product split, billing confusion | No | Reader, not a production pipeline |
| WellSaid / Murf | No (TTS only) | Basic | $19–$160/mo | No | Corporate/e-learning, not fiction |
| DeepZen | No (you upload manuscript, they TTS) | Limited (managed tier) | $69–$129/finished-hr | No | Historically "cannot distinguish between characters" |
| Resemble AI | ✅ Yes (STS, one take → N voices) | Yes | Enterprise API; target voice ≥10 min | No (API lock-in) | Closest commercial analogue — no gates exposed |
| Respeecher | ✅ Yes | Yes | Custom enterprise | No | Sound-engineer-heavy managed service |
| Azure AI Speech (VC) | ✅ Yes — "preserves source prosody", 91 locales | Yes | Cloud API, per-char/min | No | Confirms enterprise demand for prosody-preserving VC |
| Seed Audio 1.0 (ByteDance) | ❌ No (TTS from prompts) | Yes | Credit packs | No | Newest entrant; validates long-form consistency pain, not S2S |
| Alexandria (OSS, ~990★) | ❌ No (Qwen3-TTS) | Yes (full cast, LoRA) | $0 | ✅ Docker | Best OSS *TTS*; donor performance still lost |
| audiobook-creator (OSS, ~471★) | ❌ No (Kokoro/Orpheus) | Yes (emotion tags) | $0 | ✅ | TTS-from-text path |

### Where commercial & OSS fall short (evidence-backed)
- **Long-form voice consistency is the #1 documented pain even at the leader**: ElevenLabs staff admit v3 "expressiveness vs consistency" trade-off and advise Multilingual v2 "for long-form audiobook"; users report "same voice, paragraph to paragraph, drastically changes" and credits burned fighting it (r/TextToSpeech 1rzj5pr, r/ElevenLabs 1s8cmdl).
- **Pitch/level seams at chunk joins are measurable**: an audiobook producer measured **30–48 Hz pitch jumps at joins** (~1 semitone; ~20 Hz is listener-noticeable), needing request-stitching + take-scoring + objective QA. A continuous-source STS pipeline avoids this structurally.
- **Pacing/emotion loss is the recurring critique of TTS-from-text narration**: "lacks the nuanced emotional expression a human actor provides" (ElevenLabs' own 2023 guide); HN: "narrator changes key… pacing, emotion None of that is there" (news.ycombinator.com 42708773, 34256592).
- **Character drift**: DeepZen "cannot distinguish between characters" (KDP forums); r/aitubers: "characters drift, pacing gets weird, you end up babysitting it".
- **Cost + lock-in**: LOVO Chapter-7 liquidation (May 2026) and Play.ht migration guides are real vendor-risk precedent; "use-it-or-lose-it" credits (r/TextToSpeech 1sf5n70).

---

## 4. Where the market stands

1. **Multi-voice audiobook generation has commoditized — but ONLY as TTS-from-text.** ElevenLabs, Play.ht, Speechify, DeepZen, Alexandria, audiobook-creator all re-speak the book and **discard the narrator's recorded performance**. That is exactly where reviewers and HN say AI narration fails.
2. **Speech-to-speech "convert my existing narration into N character voices" exists only at two extremes**: hosted cloud APIs (ElevenLabs STS, Resemble, Respeecher, Azure VC — locked, credit-metered, no objective gates) OR raw OSS converters (Seed-VC/RVC — free, self-hosted, but no audiobook shape, no gates, no preset reuse).
3. **Nothing sits in the middle**: self-hosted STS **plus** objective gating **plus** reusable per-character presets. That middle is open.
4. **Usage reality**: the dominant audiobook spaces on HF are XTTS-based (re-speak); the dominant S2S engine (Seed-VC) is **archived upstream with a GPL-3.0 license** — the fork + license decision is ours to make now.

---

## 5. Gap analysis → our defensible position

| Market need (evidence) | Commercial today | OSS today | Our slot |
|---|---|---|---|
| Keep the narrator's performance (pacing/emotion) | Only paid STS APIs, per-min-metered | Seed-VC fork (zero-shot) | Gated self-hosted STS |
| Character consistency across long chapters | Known failure (v3 seams, 30–48 Hz) | None address it | Fixed donor + fixed per-char reference/preset + pass/fail gates on every output |
| Objective, pre-committed acceptance | None expose gates | None | DNSMOS P.835 + UTMOS + pacing ±5-8% + f0 + blind A/B |
| Reusable per-book config | Locked in subscriptions | per-voice ad-hoc | Shipped presets |
| Grok "is it any good" without paying first | Pay-per-revision | Manual trial-and-error blogs | One-chapter proof with gate-scored numbers |

---

## 6. Initial step (recommended)

> **Build the one-chapter proof on a fork of Seed-VC V2.**
> Take one clean recorded chapter → convert to 4 preset timbres (2 fiction, 1 non-fiction, 1 motivational)
> → EQ/de-ess/loudness-normalize → gate every output (DNSMOS P.835 OVRL≥3.6-3.8, UTMOS≥3.8-4.0,
> pacing ±5-8% vs source, f0, blind A/B) → export each character as a reusable preset.
> Hardware: existing 2×T4 (Kaggle). Fallback: RVC WebUI (MIT) for any voice that needs per-voice fine-tune lock-in.

**Why first:**
- Attacks the two pain points evidence shows every competitor fails — pacing/emotion loss (STS preserves the donor's performance; TTS-from-text destroys it) and long-form consistency (fixed donor + fixed reference + ticked gates vs. stateless chunk re-synthesis that demonstrably drifts 30–48 Hz).
- Buildable in a few days, zero new hardware cost.
- Produces a concrete gate-scored artifact — one chapter, four stable voices with pass/fail numbers — **no competitor can claim as a self-hosted, presets-reusable, gated speech-to-speech converter**.

**Blocking decisions to make before / during the build:**
1. **License**: Seed-VC is **GPL-3.0** (fork stays GPL unless we reimplement/abstract; its weights pull the S2S property). RVC/Applio are MIT but lose zero-shot pacing preservation. Decide product intent (self-use vs redistributing the converter) → determines primary engine.
2. **Fork maintenance**: seed-vc is archived upstream → fork first; pin `diffusion-steps`, `f0-condition=False` for prose, config presets per character in the fork.
3. **Need a 1-2 model warm-up POC on Kaggle** (2×T4) to confirm Seed-VC V2 fp16 runs + one chapter end-to-end before committing the repo structure.

---

## Sources
- HF: Plachta/Seed-VC · lj1995/VoiceConversionWebUI · IAHispano/Applio · xihajun/GPT-SoVITS · myshell-ai/OpenVoiceV2 · fishaudio/fish-speech-1.4 · coqui/XTTS-v2 · ResembleAI/chatterbox · spaces jkorstad/AudioBook & ebook2audiobook variants
- GitHub: Plachtaa/seed-vc (archived, ~3.9k★, GPL) · RVC-Project/Retrieval-based-Voice-Conversion-WebUI (~38k★, MIT, 2026-08 push) · IAHispano/Applio
- Papers: Ref-VC arXiv 2508.04996 (SOTA over Seed-VC on noisy zero-shot; no HF release yet — watch, don't bet)
- Market: ElevenLabs pricing/trust pages, reedsy.com/blog/elevenlabs-review, lex-books.com AI-voice-consistency-seams, reddit r/TextToSpeech (1rzj5pr, 1sf5n70), r/ElevenLabs (1s8cmdl), r/aitubers (1r6x7eo), news.ycombinator.com (42708773, 34256592), KDP community forums (DeepZen), LOVO liquidation (May 2026) coverage