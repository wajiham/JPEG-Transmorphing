# Privacy-Preserving Image Sharing

**Semester Research Project | Image Privacy, Facial Identity Protection & Generative AI**

## Overview

Sharing images online can expose sensitive visual information, including a person's facial identity. Advances in generative AI make it increasingly important to investigate how shared images can be protected against unauthorized manipulation and identity misuse.

This repository documents an ongoing semester research project exploring **privacy-preserving image transformations** and protection against AI-based image manipulation. The work currently focuses on building technical foundations, reviewing literature, comparing protection mechanisms, and identifying research gaps before selecting a final approach.

> **Status:** Literature review and research problem formulation in progress. No final protection method or experimental results are claimed yet.

## Research Questions

- How can facial identity be protected when images are shared online?
- How do existing methods prevent or reduce unwanted AI-based image editing or identity manipulation?
- How robust are protection mechanisms to JPEG compression and other image processing operations?
- What trade-offs exist between privacy, visual quality, robustness, and practicality?

## Literature Review
The reviews examine each work's problem, central idea, protection target, methodology, strengths, weaknesses, and relevance to the semester project.

## Repository Structure

- [Background](01-background/)
- [Literature Review](02-literature-review/)
- [Research Analysis](03-research-analysis/)
- [Experiments](04-experiments/)


## Research Workflow

1. **Technical foundations** — Understand JPEG encoding, image representations, compression, and transformations.
2. **Literature review** — Analyze existing methods for facial identity and image privacy protection.
3. **Comparison and gap analysis** — Identify differences in threat models, assumptions, strengths, and limitations.
4. **Research direction** — Formulate a focused research question and candidate approach.
5. **Experimental evaluation** — Select baselines, models, datasets, and metrics, then test the approach.

## Potential Evaluation Dimensions

- **Identity privacy:** How well a defense protects against a clearly specified identity-related threat.
- **Image quality:** Whether protected images remain visually acceptable to humans.
- **Robustness:** Whether protection survives JPEG compression, resizing, and related transformations.
- **Generative AI behavior:** Whether the selected manipulation or editing system is affected.
- **Practicality:** Computational cost and suitability for user-facing workflows.

The precise evaluation protocol will be defined after the research question is finalized.

## Project Status

This repository is a **work in progress**. Literature notes document existing research and do not indicate that those methods have already been implemented, reproduced, or experimentally validated within this project.

