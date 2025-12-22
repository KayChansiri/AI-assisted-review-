# AI-Assisted Systematic Review Workflow

This repository contains the full workflow, code, and experimental prompt designs for an AI-assisted systematic review pipeline optimized for **abstract-level screening with false-negative minimization**. The project demonstrates how large language models (LLMs) can be deliberately tuned—via prompt design rather than model retraining—to operate at different stages of a systematic review.

Author: Karikarn (Kay) Chansiri, Ph.D.

---

## Project Overview

Systematic reviews at the abstract screening stage prioritize **recall (sensitivity)** over precision to avoid missing relevant studies. However, default LLM prompting often mirrors *full-text eligibility logic*, which suppresses recall and increases false negatives.

This project develops and evaluates multiple prompt strategies that explicitly encode:

* different **decision objectives** (final inclusion vs. triage),
* different **loss functions** (false-negative vs. false-positive cost), and
* different **inferential envelopes** for ambiguous abstracts.

The workflow is divided into **Phase 1 (development + alignment)** and **Phase 2 (validation)**, using human-labeled data as the reference standard.

---

## Repository Structure

```
├── tested_prompt_phase1/
│   ├── Phase1_merged.xlsx
│   ├── Phase1_with_FNreduction_temp0.xlsx
│   ├── Phase1_strategy_2.1_refined.xlsx
│   ├── Phase1_strategy_2.1_refined_final.xlsx
│   ├── Phase1_with_humans_updated.xlsx
│   └── analysis notebooks
│
├── to_betested_prompt_phase1/
│   ├── Phase1_test.xlsx            # small test dataset used in code examples
│   └── prompt testing scripts
│
├── phase2/
│   ├── Phase_2_combined.csv
│   └── Phase2_AI_screening_results.csv
│
└── README.md
```

GitHub serves as the primary worksite for prompt iteration, model testing, and metric comparison.

---

## Model

Current model used in the experiments:

* **gpt-4o**

The code is model-agnostic and can be adapted to newer OpenAI models or external LLMs (Claude, LLaMA, Gemini).

To update the model:

1. Visit the OpenAI model documentation or Hugging Face.
2. Select the desired model.
3. Copy the model identifier.
4. Replace the model name in the API call within the provided scripts.

---

## Phase 1: Prompt Design and Alignment

### Objective

Evaluate how different prompt formulations shift the tradeoff between recall and precision during abstract screening.

### Prompt 1: Full Eligibility Determination (Zero-Shot)

This prompt asks the model to behave like a **full-text reviewer**:

* evaluates 8 independent eligibility criteria,
* applies a hard AND-gate logic,
* assigns explicit exclusion codes,
* makes a final inclusion decision.

Logical structure:

Include = C1 ∧ C2 ∧ C3 ∧ … ∧ C8

This structure strongly suppresses recall because abstracts often underspecify intervention details, implementation, effectiveness, or geography.

Performance:

* Precision: 0.851
* Recall: 0.430
* F1: 0.571
* AUC: 0.706

Primary issue: **Low recall (false-negative inflation)**.

---

### Prompt 2: Binary Triage with False-Negative Minimization (Few-Shot)

This prompt reframes the task as **pre-screening**, not final inclusion:

* reduces criteria from 8 to 4,
* removes effectiveness, implementation, and geography from exclusion logic,
* excludes only when studies are clearly incompatible.

Implicit decision rule:

Include if (Youth ∧ Possibly Homeless ∧ Some Intervention ∧ Empirical)

This better approximates human title/abstract screening heuristics.

Performance:

* Precision: 0.405
* Recall: 0.735
* Specificity: 0.648
* Accuracy: 0.670
* F1: 0.522

---

### Prompt 3: Expanded Inferential Envelope (Maximum Recall Regime)

This prompt explicitly corrects known human-AI misalignment cases:

* mixed-age samples where youth are likely present,
* housing-related interventions with non-housing outcomes,
* systems-level homelessness research,
* public-health interventions in homeless-serving contexts.

Key design principles:

* vulnerability → homelessness risk inference allowed,
* systems-level contexts allowed,
* exclusion only if explicitly irrelevant.

Decision boundary shifts from:

“Does this clearly match youth homelessness criteria?”

to:

“Is it plausible this intersects youth homelessness systems or populations?”

Performance (Phase 1):

* Precision: 0.371
* Recall: 0.892
* Specificity: 0.616
* Accuracy: 0.672
* F1: 0.524

---

## Phase 2: Validation

### Data

* Phase_2_combined.csv

### Prompt Used

* Prompt 3 (expanded inferential envelope)

### Performance

* Recall: 0.914
* Specificity: 0.493
* Precision: 0.206
* Accuracy: 0.546
* F1: 0.336

These results confirm that Prompt 3 generalizes as a **high-sensitivity screening tool**, suitable for early review stages.

---

## Key Findings

* LLMs can be tuned via prompt design to favor recall or precision.
* False-negative minimization is achievable without model retraining.
* Different prompt regimes correspond to different systematic-review stages.
* Prompt 3 performs best when the goal is **not to miss relevant studies**.

---

## Ongoing and Future Work

Planned extensions include:

* testing zero-shot vs. few-shot prompting under identical decision structures,
* comparing direct decision vs. chain-of-thought reasoning,
* binary vs. Likert decision outputs,
* supervision regime comparisons,
* cross-model benchmarking (Claude, LLaMA, Gemini),
* cost monitoring to keep total API usage under $300 per experiment.

---

## Notes

* Code examples use `Phase1_test.xlsx` as a small test dataset.
* Full Phase 1 experiments should be run on `Phase1.xlsx`.
* File names should be updated accordingly when running full-scale analyses.

---

## Citation

If you use or adapt this workflow, please cite or acknowledge this repository.
