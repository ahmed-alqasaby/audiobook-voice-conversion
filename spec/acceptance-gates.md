# Acceptance Gates — the pre-committed contract

Every character render produced by this pipeline must pass the numeric gates below.
The numbers in this file are **the contract**: they are fixed before audio is generated,
and they do not move to fit a result. A failed gate routes to a retune or a different
engine — never to editing this page.

Gate order matters: the POC runs the "fast core" (DNSMOS, UTMOS, pacing, length) first
and only spends compute on the rest after those pass.

## The gates

| Gate | Measures | Formula / method | Threshold | Tool |
|---|---|---|---|---|
| DNSMOS P.835 | Clean signal, no artifact (no reference needed) | No-reference NN → SIG / BAK / OVRL (1–5) from 4 s windows @16 kHz | **OVRL ≥ 3.6**, SIG ≥ 3.8 | `microsoft/DNSMOS` (`.onnx`) |
| UTMOS | Naturalness ("human, not robotic") | No-reference MOS predictor, ensemble of SLT features | **≥ 3.8** (target 4.0); < 3.5 ⇒ artifact | `tarepan/SpeechMOS` |
| Pacing | Speed of delivery survived conversion | Speaking rate = syllables/sec via forced alignment; render within 0.92–1.08× donor on same text | **±5%** target, **±8%** hard limit | MFA / Whisper timestamps |
| Length ratio | Cheap pacing proxy | `render_duration ÷ donor_duration` (`--length-adjust 1.0`) | **0.95–1.05** | ffprobe |
| f0 drift | Pitch didn't flatten/jump | `12·log₂(f₂/f₁)` on per-frame F0 (librosa `pyin` / RMVPE), donor vs render | **≤ ±1.5 st** | librosa / RMVPE |
| SECS | Timbre is the character, not the donor | WavLM-style speaker-embedding cosine similarity, render vs **reference clip** | **calibrated on POC set** (below) | WavLM-based SECS |
| Loudness | Level + headroom | EBU R128 / ITU BS.1770 integral + true peak | **−16 ± 1.5 LUFS**, TP ≤ −1.5 dBTP | ffmpeg `loudnorm` / pyloudnorm |
| Blind A/B | Human sanity net over all numbers | Unlabeled listener pass: donor + renders, flag artifacts/naturalness | pass on small panel | manual |

## SECS calibration (do not hard-code from the literature)

SECS thresholds are **encoder-specific** — numbers from papers (~0.7–0.9) do not transfer
to our chosen encoder. The POC must: (1) embed donor vs renders and donor vs references on
a small measured set, (2) pick the SECS floor that separates "sounds like the character"
from "sounds like the donor", (3) record that floor **here** before treating it as a gate.

## What a failed gate means

1. Retune the voice (different reference clip / conversion params) and re-run — one attempt.
2. If still failing: escalate that voice to the RVC fine-tune engine (ADR-0001 winner logic applies per voice).
3. Publish (showcase bar) requires **all 4 voices passing every gate** — a failed voice blocks publishing, not thresholds.

## References

Thresholds derived from `research/vc-emotion-prosody.md` §3 and validated in
`research/ecosystem-map.md` — real-repo ecosystem scan.