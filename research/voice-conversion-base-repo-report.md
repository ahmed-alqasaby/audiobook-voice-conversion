# Voice Conversion Base Repo Selection — Research Report

**Project:** Self-hosted audiobook voice-conversion pipeline — one reference narration → 4 distinct character voices via speech-to-speech (S2S) timbre conversion that preserves pacing & prosody (NOT TTS re-speaking).
**Constraint:** 2× T4 GPUs. **Gate:** pacing Δ within ±5–8% vs source; DNSMOS P.835, UTMOS, f0, blind A/B metrics.
**Date:** 2026-09-04. **Status:** Research complete; recommendation firm.

---

## Comparison table

| Repo | Stars | Forks | Last commit | Active/Archived | License | Package (CLI/Docker/pip/UI) | Fits our use? | Notes |
|---|---|---|---|---|---|---|---|---|
| **Plachtaa/seed-vc** | 3,888 | 539 | pushed 2025-04-20; **archived 2025** | **Archived (read-only)** | **GPL-3.0** | Python scripts (no pip/Docker) + Gradio WebUI + real-time GUI | **Best S2S engine** — true zero-shot timbre VC preserving pacing, `--length-adjust`, `--f0-condition`, config presets, few-shot fine-tune | frozen upstream; must fork. Engine-fit, packaging-fit is ours |
| **RVC-Project/Retrieval-based-Voice-Conversion-WebUI** | 38,081 | 5,258 | **2026-08-04 (active)** | **Active** | **MIT** | `webui.py` Gradio, `realtime_gui.py`, Docker (`docker-run.sh`), `api_240604.py` | **Best maintained fallback** | per-voice fine-tune (≤10 min data) + retrieval index = the RVC v2 plan-B |
| **RVC-Project/Retrieval-based-Voice-Conversion** (library/API) | 614 | 133 | pushed 2025-11-05 | Active but **"in preparation..."** (develop branch) | MIT | README shows pip `rvc`, `rvc infer` CLI, `rvc-api`, Docker | **Not production-ready yet** | README explicitly: *"Currently under development... Provided as a library and API in rvc."* Do not rely on it today |
| **RVC-Boss/GPT-SoVITS** | 61.5k | 6.6k | active 2026 | Active | MIT (code) / unclear weights | Gradio WebUI + `api.py`/`api_v2.py` + Docker/compose + Colab | Partial — TTS-centric | TTS re-speaking, not S2S timbre-preserving; licensing of trained weights is murky |
| **fishaudio/fish-speech (S2 Pro)** | large, active | — | 2026 (new) | Active | **Fish Audio Research License (non-commercial)** | Docker/compose + api.py, SGLang/vLLM | **No — NC license + huge VRAM** | "Any commercial use requires a separate written license." 4–5B params >> 2×T4. Multi-speaker via `<speaker:i>` but TTS-based |
| **yxlllc/DDSP-SVC** | 2.7k | 285 | active (6.3) | Active | MIT | `main_reflow.py` CLI + GUI + batch infer | Marginal — singing-vocals focus | multi-speaker via `n_spk`, timbre mixing; prosody-preserving but low-res / singing-optimized |
| **svc-develop-team/so-vits-svc** | 28.1k | 5.0k | 2023-11-11 (archived) | **Archived** | AGPL-3.0 | `inference_main.py` CLI + webUI + Flask API | No | superseded by RVC / Seed-VC; singing focus |
| **coqui-ai/TTS (XTTS)** | 46.0k | 6.1k | Dec 2023 | **Defunct / archived** | MPL-2.0 code / **CPML weights (non-commercial)** | pip + Docker + CLI | No | company shut down Jan 2024; **no rights holder to grant commercial license**; XTTS is TTS not S2S VC |

### Audiobook / multi-character / one-narrator→N-voices projects (examined)
All of these are **LLM-annotation + TTS re-speaking**, which by definition re-times the audio and would fail the pacing ±5–8% gate. None are S2S timbre-conversion:

- `prakharsr/audiobook-creator` (523★, GPL-3.0, Gradio + Docker) — Kohler/Orpheus TTS with LLM character attribution. [src](https://github.com/prakharsr/audiobook-creator)
- `Finrandojin/alexandria-audiobook` — Qwen3-TTS multi-voice + LoRA fine-tune, per-line style. [src](https://github.com/Finrandojin/alexandria-audiobook)
- `psdwizzard/chatterbox-Audiobook` (205★) — Chatterbox TTS multi-voice, LUFS presets. [src](https://github.com/psdwizzard/chatterbox-Audiobook)
- `zackham/narrate` (15★, MIT) — Chatterbox/ElevenLabs JSONL script, `narrate.toml`. [src](https://github.com/zackham/narrate)
- `RockNCode/novelcast` — multi-voice light-novel studio. [src](https://github.com/RockNCode/novelcast)
- `debpalash/VoiceStudio` — "open-source ElevenLabs alternative," voice cloning + design + dubbing. [src](https://github.com/debpalash/VoiceStudio)

---

## Best base repo recommendation

**Build on Seed-VC as the primary S2S engine (forked), with RVC-Project/Retrieval-based-Voice-Conversion-WebUI as the maintained fallback.** Evidence is quoted from the raw READMEs (fetched 2026-09-04).

### Seed-VC — engine fit (primary)
- **True zero-shot S2S preserving pacing/prosody (the core requirement):**
  > "zero-shot voice conversion … **Without any training, it is able to clone a voice given a reference speech of 1~30 seconds.**"
  This is timbre conversion of *existing* audio — the source's pacing/prosody is carried through, which is exactly the property the pacing gate enforces (unlike GPT-SoVITS/Fish/XTTS, which re-speak via TTS).
- **First-class pacing control (`--length-adjust`):** the README documents
  > "`length-adjust` is the length adjustment factor, default is 1.0, set <1.0 for speed-up speech, >1.0 for slow-down speech"
  — a per-file rate dial directly usable to hold the Δ within ±5–8%.
- **Pitch / f0 gating (`--f0-condition`, `--auto-f0-adjust`, `--semi-tone-shift`):** documented flags cover source-conditioned pitch and semitone transposition for the f0 gate.
- **Config/preset files for reproducibility:** per-model YAML presets under `configs/presets/` (`config_dit_mel_seed_uvit_xlsr_tiny.yml`, `config_dit_mel_seed_uvit_whisper_small_wavenet.yml`, `config_dit_mel_seed_uvit_whisper_base_f0_44k.yml`, and `configs/v2/vc_wrapper.yaml`). Natural home for per-book/per-voice reproducible presets.
- **Few-shot / one-shot per-voice fine-tune:**
  > "minimum 1 utterance per speaker" … "extremely fast training speed (**minimum 100 steps, 2 min on T4**)"
  `train.py` for V1, `accelerate launch train_v2.py --train-cfm` for V2 (multi-GPU). Tunes similarity per character; can hold WER/similarity balanced.
- **V2 accent/emotion + anonymization control:** `--convert-style true` (AR model), `--similarity-cfg-rate` / `--intelligibility-cfg-rate`, `--anonymization-only`.
- **Hardware fit on 2×T4:** checkpoints are small — v1.0 tiny **25M**, offline **98M**, SVC **200M**, V2 **67M(CFM)+90M(AR)**. All comfortably fit a T4.
- **Programmatic surface:** pure-Python modules (`inference.py`, `inference_v2.py`, `seed_vc_wrapper.py`, `eval.py`) — no packaging dependency, so we own the wrapper/API layer without fighting an installed framework.

### RVC WebUI — maintained fallback / plan-B
- **MIT, actively maintained** (last push 2026-08-04, 38k★, Docker + `api_240604.py` + realtime GUI). This is the living, commercial-safe RVC.
- **Per-voice fine-tune + retrieval index** precisely matches the planned RVC v2 fallback path ("train a good VC model with data ≤10 min").
- *Correction vs. earlier draft:* the sibling **library/API repo (`Retrieval-based-Voice-Conversion`) is still marked "in preparation..."** (default branch `develop`, pushed 2025-11-05) and its README says "Currently under development." So do **not** today depend on the curated `rvc infer`/`rvc-api` package; the WebUI (or forking its `infer/` + `train/` modules) is the real surface.

### Why not the others
- **GPT-SoVITS / Fish Audio S2 / XTTS** are **TTS re-speakers** — they generate new audio from text, not preserve source pacing/prosody. They fail the core S2S-preserve-pacing requirement by design.
- **Fish Audio S2 Pro** additionally has a **non-commercial research license** ("any commercial use requires a separate written license") and **4–5B params** that don't fit 2× T4.
- **Coqui/XTTS** is defunct with **no active rights holder** to grant a commercial license for the CPML weights.
- **so-vits-svc** is archived (AGPL). **DDSP-SVC** is singing-vocals-optimized, lower quality for speech audiobook narration.
- **Audiobook tools** are all TTS-based; none do S2S timbre conversion.

### Caveats to resolve before commit
1. **Seed-VC is archived (2025).** No bug fixes, no dependency updates, torch stack pinned to ~2024-era. **Mitigation: fork as the base and own maintenance**, or swap the engine to the actively-maintained RVC WebUI for characters that need retrieval-index quality. Recommendation stands because the S2S-pacing-preserve property (the hardest requirement) is uniquely satisfied by Seed-VC and is *not* available from any maintained repo.
2. **GPL-3.0 licensing.** Any shipped/derived work must be GPL-3.0-compatible and source-disclosed. If the audiobook service is commercial, either stay internal/self-hosted (fine) or plan GPL-compliant distribution. RVC (MIT) is the commercial-safe alternative if GPL is a blocker — but you lose the true S2S pacing-preservation.
3. **Post-processing & metrics are ours.** **No base project ships** the master chain (adaptive EQ / de-ess / loudness normalize) or the DNSMOS P.835 / UTMOS / pacing-Δ% / f0 / blind-A/B harness. Seed-VC's `EVAL.md` + `eval.py` give WER/similarity but **not** P.835/UTMOS/AB. These are net-new, in-our-control value.

---

## Market stand

- **Maintenance is fractured and two-tiered.** Only **RVC WebUI** (MIT, pushed 2026-08-04) and **DDSP-SVC** are genuinely maintained. **Seed-VC (archived 2025), so-vits-svc (archived 2023), Coqui (defunct 2024)** are frozen. The last-gen VITS/so-vits stack has consolidated into RVC; the research frontier moved to Seed-VC/DiT, which is now unmaintained — a real ecosystem gap.
- **The dominant open-source effort has drifted from pure VC toward TTS** (GPT-SoVITS 61k★, Fish Audio S2, XTTS). That drift is *why nobody maintains a S2S one-narrator→N-voices audiobook tool* — the biggest open tools optimize TTS flexibility, not source-pacing-preserving S2S conversion.
- **Licensing splits the field into three buckets:**
  - **Commercial-safe (MIT):** RVC WebUI (+ library), DDSP-SVC, GPT-SoVITS code (though its trained-weight terms remain murky — verify before shipping).
  - **Non-commercial / blocked:** Fish Audio S2 Pro (research license), Coqui/XTTS (CPML, no rights holder after shutdown).
  - **Copyleft:** Seed-VC (GPL-3.0), so-vits-svc (AGPL-3.0), audiobook-creator (GPL-3.0).
- **Ownership concentration / bus factor.** One individual+org — **RVC-Boss / RVC-Project / lj1995 / yxlllc** — authors and maintains the dominant, actively-maintained tooling (RVC WebUI, RVC library, GPT-SoVITS, RMVPE). A single maintainer can walk away (Plachtaa archived Seed-VC). Architect for a fork.
- **Hardware reality favors the small model.** The newest "best-quality" models (Fish S2 ~4–5B, AR+CFM V2) exceed 2× T4. Seed-VC checkpoints (25M–200M) and RVC v2 fine-tunes run comfortably — another reason the Seed-VC + RVC stack fits the 2×T4 constraint and Fish S2 does not.
- **Quality metrics are a whitespace differentiator.** DNSMOS P.835 / UTMOS / pacing-Δ% / f0 / blind-A/B exist in **no** base project. Your gating harness is net-new, defensible value — the repo this pipeline sits on does not matter for it.
- **The exact niche is unoccupied.** Multi-voice audiobook work (Alexandria, audiobook-creator, chatterbox-Audiobook, narrate, novelcast, VoiceStudio) is uniformly **LLM-annotation + TTS re-speaking**; none preserve the narrator's original pacing/prosody via S2S timbre conversion. That property — enforced by the pacing gate — is the claimable whitespace.

---

## Sources
- Seed-VC README (raw): https://raw.githubusercontent.com/Plachtaa/seed-vc/main/README.md — zero-shot S2S, `--length-adjust`, `--f0-condition`, configs, few-shot fine-tune, model sizes, T4
- Seed-VC API: https://api.github.com/repos/Plachtaa/seed-vc — archived=true, GPL-3.0, 3,888★
- RVC WebUI README: https://raw.githubusercontent.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI/main/README.md — MIT, fine-tune ≤10 min
- RVC WebUI API: https://api.github.com/repos/RVC-Project/Retrieval-based-Voice-Conversion-WebUI — pushed 2026-08-04, 38k★, active
- RVC library README: https://raw.githubusercontent.com/RVC-Project/Retrieval-based-Voice-Conversion/main/README.md — "Currently under development"
- RVC library API: https://api.github.com/repos/RVC-Project/Retrieval-based-Voice-Conversion — description "in preparation...", develop branch, pushed 2025-11-05
- Fish Audio license: https://github.com/fishaudio/fish-speech/blob/main/LICENSE — non-commercial research license
- Coqui/X TTS: https://github.com/coqui-ai/TTS/issues/3490 and https://aiwiki.ai/wiki/xtts — defunct + CPML
- Audiobook tools: https://github.com/prakharsr/audiobook-creator, https://github.com/Finrandojin/alexandria-audiobook, https://github.com/psdwizzard/chatterbox-Audiobook, https://github.com/zackham/narrate, https://github.com/RockNCode/novelcast, https://github.com/debpalash/VoiceStudio