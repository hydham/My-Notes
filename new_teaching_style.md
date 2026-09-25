---
name: teaching-style
description: Standing spec for how Hydham wants technical topics taught in this project, plus session structure
sources: [backfill]
aliases: [teaching-style.md]
---
- [stated] Treat the following as a standing teaching spec for new topics in this project
- [stated] Concepts before syntax — introduce every new term or resource only after the preceding concept creates a visible gap that needs filling; assume no prior knowledge of the specific topic being taught
- [stated] Treat Hydham as a complete beginner with zero prior knowledge — not a developer; do not assume familiarity with programming, developer/production tooling conventions, or jargon (e.g. TLS, git workflows); build every such term from the ground up before using it
- [stated] Visual/diagram content over prose — ASCII diagrams woven into the narrative as the primary explanatory vehicle, not appended afterward; minimize wall-of-text prose explanations
- [stated] Code and syntax blocks must include detailed inline comments — every non-obvious line commented directly in the block, rather than relying on a separate prose section after the block to carry the explanation
- [stated] Consistent metaphors — if a chapter-wide metaphor is used (e.g. "empty plot of land" for VPC), apply it to every subsequent concept in that chapter; otherwise avoid unrelated-domain analogies
- [stated] Shorter response chunks with comprehension checks — deliver one topic at a time and confirm understanding before proceeding; do not front-load a full section in one response
- [stated] Gap-filling over transcript following — explicitly address motivations and sequencing the source material skips; prefer chain-of-causation reasoning over summary
- [stated] Include self-check prompts at the end of each section
- [stated] Notes that introduce syntax before the underlying concept is established break Hydham's learning flow — this happened once in a Terraform session and the affected section had to be fully rebuilt

## Session structure
- [stated] Hydham often brings a transcript or tutorial as source material and asks Claude to teach from it with the above spec applied
- [stated] Corrections are direct and specific; accept them and rebuild the explanation rather than defending the original response