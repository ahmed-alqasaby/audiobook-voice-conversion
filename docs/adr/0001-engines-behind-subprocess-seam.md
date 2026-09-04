# ADR-0001: Engines run as external subprocesses; pipeline code is MIT

We run our own conversion/inference engines (Seed-VC, RVC) as **external, unmodified
processes invoked via subprocess — never imported into or vendored inside our package** —
so our pipeline code stays **MIT** and clean, publishable even if an engine is GPL
(Seed-VC is GPL-3.0; RVC/Applio are MIT). A GPL engine imposes no obligation on sold
outputs and no source disclosure when software is never conveyed (SaaS/ASP: Seed-VC is
GPL, not AGPL), provided the process boundary is respected.

**Status:** accepted · **Considered:** importing Seed-VC in-process (taints our MIT code
under GPL-3.0 linking); GPL-licensing the whole repo (hurts the MIT showcase posture);
MIT-only (forecloses Seed-VC's zero-shot pacing-preserving speech-to-speech).

**Consequences:** users of the published pipeline install engines themselves as documented
dependencies; the per-voice engine choice is an **empirical decision** (`spec/poc.md`) —
Seed-VC zero-shot first, RVC fine-tune only when gates fail; the winner per voice becomes
the default and is recorded in that voice's preset, not in this ADR.