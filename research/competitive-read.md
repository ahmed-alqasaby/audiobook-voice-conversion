# Competitive read — proposals on job 40688048

8 proposals reviewed (2026-09-04). The field is **uniformly qualitative**:

| # | Proposal style | Engine | Numeric gates? | Self-hosted model? | Thesis |
|---|---|---|---|---|---|
| 1 | "5 yrs voiceover/audio production" | ElevenLabs + similar | no | no | generic, offers chapter test |
| 2 | "My dear prospective client" (long) | none (Adobe/Pro Tools/Audacity) | no | no | admits no neural-TTS expertise; fluff; weakest |
| 3 | "6+ yrs AI automation" | ElevenLabs + similar | no | no | model selection + presets, vague |
| 4 | "One narration, four characters… mastering problem" | ElevenLabs + fine-tune | no | no | best tone: gets mastering, spectral repair, de-ess, one-chapter proof |
| 5 | "24–48h live sample" | (unstated) | no | no | action/speed focused |
| 6 | "5+ yrs AI dev" (Shailender) | ElevenLabs + similar | no | no | solid breadth: preprocessing, loudness, artifacts, docs |
| 7 | "This looks straightforward" + portfolio link | (unstated) | no | no | portfolio-driven |
| 8 | "12+ yrs Full Stack Mobile/Web" | (unstated) | no | no | off-target |

## Patterns & the opening
1. **All 8 default to a hosted engine (ElevenLabs) or are engine-vague.** None propose
   a specific open-source, self-hosted VC model.
2. **None pre-commit a numeric acceptance threshold.** The client's two hardest
   bars — "no robotic/metallic" and "consistent pacing/volume" — are all handled
   qualitatively, so acceptance stays open to dispute after generation.
3. **None name the emotional/motivational voice (the real risk center) as a
   distinct, quantified problem.** #4 and #6 gesture at it via post-processing.
4. **No one mentions emotion/prosody *retention* (the conversion's core risk),**
   only "consistency."

## Our differentiation (gaps nobody else fills)
- **Self-hosted, reusable VC** (Seed-VC V2 on 2×T4 + RVC fallback) instead of
  a hosted-subscription lock-in → we ship weights/presets the client owns.
- **Pre-committed numeric pass/fail gates** (ticket 03) put into the proposal:
  DNSMOS/UTMOS, −16 LUFS ±1.5 / TP ≤ −1.5, pacing ±5%, f0 ±1.5 st, voice-D
  emotion ratios + blind A/B. No competitor stakes a number.
- **Explicit voice-bible-first method** and naming the emotional voice as the
  hardest — signals we've thought about the actual failure mode.
- Bid = substance + one provable claim (run one chapter through the gates),
  short and concrete, opposite of #2's word salad and #4's polish-without-numbers.
