# Participant-Centric Evidence Consolidation for Dyadic Conversational Anomaly Detection with Multimodal LLMs

This repository contains the experimental code and supporting material for an MSc dissertation on **dyadic conversational anomaly detection with multimodal large language models (MLLMs)**.

The project studies whether two individually plausible participant streams form a coherent audiovisual interaction. Three controlled conversational anomalies are considered:

- **Wrong Partner** — participants from different conversations are paired.
- **Lag Partner** — Participant B's VAD-derived speaking timeline is shifted by +2 s or +3 s.
- **Silent Partner** — an active participant is paired with a dataset-native participant record with no meaningful speech evidence.

The work uses **Qwen2.5-Omni-7B** and the **Seamless Interaction** dataset. The central finding is that participant-centric decomposition makes the task substantially more tractable, but reliable unified reasoning depends not only on whether evidence is available, but on **how semantic, temporal, and participation evidence is represented, consolidated, and calibrated**.

---

## Research Questions

The dissertation addresses four main questions:

1. **RQ1 — Direct inference:**  
   Can an off-the-shelf audiovisual MLLM distinguish NORMAL from anomalous dyadic conversations through direct zero-shot inference?

2. **RQ2 — Participant-centric decomposition:**  
   Does independently analysing each participant and exposing semantic, temporal, and participation evidence improve detection across different anomaly mechanisms?

3. **RQ3 — Evidence consolidation:**  
   Does structured consolidation produce independently utilised reasoning branches, or do semantic, temporal, and participation signals interfere with one another?

4. **RQ4 — Targeted adaptation:**  
   Can lightweight parameter-efficient fine-tuning improve semantic evidence utilisation while preserving temporal, participation, structured-output, and final classification behaviour?

---

## Method Overview

The project moves from direct end-to-end inference to a participant-centric evidence-consolidation pipeline.

```text
Raw dyadic audiovisual interaction
            ↓
Direct split-screen baseline
            ↓
Participant-centric decomposition
            ↓
┌───────────────────────────────────────┐
│  Semantic evidence                   │
│  Temporal evidence                   │
│  Participation evidence              │
└───────────────────────────────────────┘
            ↓
Cached structured evidence database
            ↓
Unified evidence consolidation
            ↓
Structured R1 + controlled ablations
            ↓
Semantic-policy refinement
            ↓
Setup F1
            ↓
Targeted semantic QLoRA
            ↓
Final source-disjoint evaluation
```

### Semantic evidence

Each participant is analysed independently over two aligned 60-second segments.

Two semantic resolutions are used:

- **Coarse summaries** — broad speech-content and topic information.
- **Focused summaries** — more specific topic, detail, specificity, and uncertainty information.

### Temporal evidence

Temporal reasoning is based on participant-specific VAD intervals and deterministic feature extraction.

The selected temporal representation retains:

- local A→B response-offset statistics;
- complete global alignment/correction features;
- a frozen NORMAL temporal reference.

Explicit filtered-turn lists and overlap features are removed from the final selected configuration.

### Participation evidence

Participation is grounded in processed VAD evidence through the participant-level:

```text
speaks
```

field.

This prevents visible facial or gestural activity from being incorrectly interpreted as evidence of actual speech.

---

## Dataset and Experimental Split

Experiments use the public **Improvised test split** of the Seamless Interaction dataset.

After structural integrity checks and speech-activity eligibility filtering:

```text
206 complete dyads
        ↓
167 eligible source interactions
```

The 167 eligible interactions are divided into:

```text
50 source conversations  → frozen temporal-reference pool
100 source conversations → development / consolidation pool
17 source conversations  → final source-disjoint evaluation
```

The 100 development sources produce a common balanced consolidation database:

```text
100 NORMAL
100 LAG
100 WRONG PARTNER
100 SILENT PARTNER
------------------
400 development cases
```

The 17 final unseen sources produce:

```text
17 NORMAL
17 LAG
17 WRONG PARTNER
17 SILENT PARTNER
-----------------
68 final source-disjoint cases
```

The final 17 source conversations are not used for temporal-reference estimation, prompt development, ablations, Setup F1 selection, fine-tuning, or model selection.

---

# Experimental Progression

## 1. Direct Monolithic Baseline

The initial system performs direct end-to-end inference on horizontally concatenated participant videos.

This baseline is unreliable and strongly biased, motivating an architecture-informed redesign.

A key issue is participant attribution: the model must simultaneously separate speakers, associate speech with visible participants, track timing, compare semantic content, and judge conversation-level coherence.

---

## 2. Isolated Participant-Centric Evidence Branches

Each evidence branch is first studied independently.

### Semantic branch

| Representation | NORMAL | Wrong Partner | Accuracy |
|---|---:|---:|---:|
| Coarse only | 91/100 | 78/100 | 84.5% |
| Focused only | 47/100 | 100/100 | 73.5% |
| Coarse + focused | 78/100 | 95/100 | **86.5%** |

### Temporal branch

| Case family | Correct | Total | Accuracy |
|---|---:|---:|---:|
| NORMAL | 83 | 100 | 83.0% |
| LAG +2 s | 41 | 50 | 82.0% |
| LAG +3 s | 47 | 50 | 94.0% |
| **Overall** | **171** | **200** | **85.5%** |

### Participation grounding

| Configuration | NORMAL | Wrong Partner | Silent Partner |
|---|---:|---:|---:|
| Coarse only | 24/30 | 30/30 | 7/30 |
| Coarse + `speaks` | 28/30 | 30/30 | 28/30 |

These experiments show that strong anomaly-specific evidence exists before unified consolidation.

---

## 3. Structured Evidence Consolidation

Participant summaries and temporal features are generated and cached **before** consolidation.

The final reasoner therefore operates only over structured textual evidence. This allows different consolidation strategies to be compared using identical underlying observations.

Four output formats are evaluated:

| Format | NORMAL | LAG | Wrong | Silent | Accuracy |
|---|---:|---:|---:|---:|---:|
| Binary only | 72 | 54 | 95 | 100 | 80.25% |
| **Structured R1** | **78** | **80** | **88** | **100** | **86.50%** |
| Free-form rationale | 90 | 43 | 46 | 100 | 69.75% |
| Structured R1 + rationale | 60 | 94 | 93 | 100 | 86.75% |

Structured R1 is retained as the main diagnostic consolidation format because it combines balanced prediction with inspectable evidence assessments.

Structured fields are treated as **diagnostic reports of observable model behaviour**, not as guaranteed faithful traces of the model's hidden reasoning process.

---

## 4. Evidence-Branch Ablations

Controlled ablations test how the Structured R1 reasoner uses its evidence.

### Participation branch

| Configuration | Accuracy |
|---|---:|
| Full Structured R1 | 86.50% |
| No participation | 75.75% |
| Filtered turns only | 87.75% |
| **`speaks` only** | **89.25%** |

### Local temporal branch

Starting from the `speaks`-only configuration:

| Configuration | Accuracy |
|---|---:|
| No local temporal branch | 83.75% |
| **Offsets only** | **91.25%** |
| Overlap only | 82.75% |

### Global temporal branch

Starting from `speaks` + offsets:

| Configuration | Accuracy |
|---|---:|
| Full global branch | **91.25%** |
| No global temporal branch | 83.50% |
| No correction magnitude | 74.50% |
| No alignment gain | 89.00% |
| No reliability evidence | 89.25% |

### Semantic branch

Removing only the semantic branch produces one of the clearest examples of **cross-branch interference**:

| Configuration | NORMAL | LAG | Wrong | Silent | Accuracy |
|---|---:|---:|---:|---:|---:|
| With semantics | 76 | 93 | 96 | 100 | 91.25% |
| No semantics | 47 | 99 | 99 | 100 | 86.25% |

Despite unchanged temporal evidence, removing semantics makes the reasoner much more anomaly-sensitive and sharply reduces NORMAL preservation.

This shows that semantic evidence is not acting only as an isolated semantic detector; it also calibrates how other evidence is interpreted.

---

## 5. Semantic Policy Refinement and Setup F1

The maximum-predictive configuration reaches 91.25% accuracy, but its structured outputs reveal substantial **semantic under-utilisation**.

The semantic decision policy is therefore revised so that semantic compatibility is judged independently from participation and temporal evidence.

| Configuration | N | L | W | S | Accuracy | WP `INCOMPATIBLE` |
|---|---:|---:|---:|---:|---:|---:|
| Max-predictive | 76 | 93 | 96 | 100 | 91.25% | 10 |
| Revised semantic policy | 84 | 74 | 90 | 100 | 87.00% | 40 |
| **Setup F1** | **75** | **85** | **96** | **100** | **89.00%** | **39** |

Setup F1 removes the verbose:

```text
detailed_speech_summary
```

while retaining coarse summaries and the compact focused semantic fields.

It is selected because it preserves strong classification performance while maintaining the improved semantic-assessment behaviour.

---

## 6. Targeted Semantic QLoRA

Setup F1 is frozen and used as the starting point for targeted parameter-efficient adaptation.

The research question is:

> **Can lightweight targeted fine-tuning improve the semantic fidelity of the unified Setup F1 reasoner for Wrong Partner detection while preserving its NORMAL, temporal-LAG, Silent Partner, structured-reasoning, and final-classification behaviour?**

QLoRA is applied only to the text-language component used during consolidation.

Key settings:

```text
4-bit NF4 quantisation
LoRA rank      = 8
LoRA alpha     = 16
LoRA dropout   = 0.05
Learning rate  = 2e-5
Max epochs     = 3
Preservation λ = 0.50
```

The semantic field receives targeted pseudo-supervision, while the remaining Structured R1 outputs are trained against Frozen F1 as a preservation teacher.

The source-group split is:

```text
40 source groups → training
10 source groups → validation
50 source groups → adapter-held-out evaluation
```

Silent Partner cases are excluded from gradient optimisation and retained as a preservation test.

---

# Final Source-Disjoint Generalisation

The final test uses 68 cases derived from **17 entirely source-disjoint conversations**.

## Classification

| Case family | Frozen F1 | Fine-tuned F1 |
|---|---:|---:|
| NORMAL | 11/17 (64.71%) | **15/17 (88.24%)** |
| LAG | **12/17 (70.59%)** | 10/17 (58.82%) |
| Wrong Partner | 16/17 (94.12%) | 16/17 (94.12%) |
| Silent Partner | 17/17 (100%) | 17/17 (100%) |
| **Overall** | **56/68 (82.35%)** | **58/68 (85.29%)** |

## Wrong Partner semantic assessment

| Model | COMPATIBLE | INCOMPATIBLE | LIMITED |
|---|---:|---:|---:|
| Frozen F1 | 7 | 6 | 4 |
| Fine-tuned F1 | 3 | **12** | 2 |

The fine-tuned reasoner therefore:

- improves overall source-disjoint accuracy from **82.35% to 85.29%**;
- increases NORMAL recognition from 11/17 to 15/17;
- preserves Wrong Partner recall at 16/17;
- preserves perfect Silent Partner performance;
- doubles explicit Wrong Partner `INCOMPATIBLE` assessments from **6/17 to 12/17**;
- but reduces LAG sensitivity from 12/17 to 10/17.

The result supports the targeted semantic-adaptation hypothesis while also showing that semantic and temporal behaviour remain functionally coupled.

---

# Repository Structure

```text
CONVERSATION_ANOMALY_DETECTION/
│
├── 01_data_preparation/
│   └── 01_download_and_preprocess_seamless_interaction.ipynb
│
├── 02_direct_baseline/
│   └── 01_direct_monolithic_video_only_baseline.ipynb
│
├── 03_isolated_branches/
│   ├── 01_semantic_normal_vs_wrong_partner.ipynb
│   ├── 02_temporal_full_development_pipeline.ipynb
│   ├── 03_temporal_selected_representation_reported_isolated_result.ipynb
│   └── 04_participation_branch_development.ipynb
│
├── 04_consolidation/
│   ├── 00_build_consolidation_evidence_database.ipynb
│   ├── 01_binary_only_consolidation.ipynb
│   ├── 02_structured_r1_consolidation.ipynb
│   ├── 03_free_form_rationale_consolidation.ipynb
│   └── 04_structured_r1_with_free_form_rationale_consolidation.ipynb
│
├── 05_ablations/
│   ├── participation/
│   │   └── 01_participation_branch_ablation.ipynb
│   ├── temporal/
│   │   ├── 02_local_temporal_branch_ablation.ipynb
│   │   └── 03_global_temporal_branch_ablation.ipynb
│   └── semantic/
│       └── 04_semantic_branch_ablation.ipynb
│
├── 06_setup_f1/
│   ├── 01_semantic_policy_refinement.ipynb
│   └── 02_setup_f1_semantic_payload_ablation.ipynb
│
├── 07_finetuning/
│   └── 01_targeted_semantic_qlora_finetuning.ipynb
│
├── 08_final_source_disjoint_evaluation/
│   ├── 01_build_final_unseen_source_disjoint_database.ipynb
│   └── 02_frozen_vs_finetuned_final_source_disjoint_evaluation.ipynb
│
├── report/
│   └── Final_Report.pdf
│
├── README.md
├── REPRODUCIBILITY.md
├── requirements.txt
└── .gitignore
```

---

## Repository-to-Thesis Mapping

| Repository stage | Thesis section | Purpose |
|---|---|---|
| `01_data_preparation` | III-B / III-D | Dataset download, preprocessing, integrity checks |
| `02_direct_baseline` | IV-A / IV-B | Direct monolithic audiovisual baseline |
| `03_isolated_branches` | IV-C / V-A–C | Semantic, temporal, and participation evidence development |
| `04_consolidation` | IV-D / V-D | Cached evidence database and consolidation-format comparison |
| `05_ablations` | IV-E / V-E–F | Feature selection, branch utilisation, and cross-branch interference |
| `06_setup_f1` | IV-E / V-G | Semantic-policy refinement and Setup F1 selection |
| `07_finetuning` | IV-F / V-H | Targeted semantic QLoRA |
| `08_final_source_disjoint_evaluation` | V-I | Final unseen generalisation test |

---

# Reproducing the Experiments

The notebooks are organised in the order in which the experimental pipeline was developed.

A simplified execution flow is:

```text
01_data_preparation
        ↓
03_isolated_branches
        ↓
04_consolidation/
00_build_consolidation_evidence_database.ipynb
        ↓
consolidation-format experiments
        ↓
05_ablations
        ↓
06_setup_f1
        ↓
07_finetuning
        ↓
08_final_source_disjoint_evaluation
```

Several notebooks depend on:

- large audiovisual files from Seamless Interaction;
- cached semantic summaries;
- cached temporal features;
- frozen temporal-reference files;
- generated JSON databases;
- the Qwen2.5-Omni-7B checkpoint;
- a saved QLoRA adapter.

These large artifacts are not necessarily distributed directly with this repository.

The notebooks are therefore intended to provide a transparent record of the complete experimental pipeline and the code required for reproduction when the corresponding data and intermediate artifacts are available.

They are **not expected to run independently without the required dataset, cached files, model checkpoint, and path configuration**.

For detailed:

- dataset requirements;
- expected directory structure;
- notebook execution order;
- required input artifacts;
- generated output artifacts;
- model/checkpoint requirements;
- hardware considerations;
- path configuration;

see:

**[REPRODUCIBILITY.md](REPRODUCIBILITY.md)**

---

# Requirements

Core dependencies are listed in:

**[`requirements.txt`](requirements.txt)**

The project relies primarily on:

- Python
- PyTorch
- Hugging Face Transformers
- Qwen2.5-Omni
- PEFT / LoRA
- bitsandbytes
- NumPy
- pandas
- scikit-learn
- SciPy

GPU execution is strongly recommended for Qwen2.5-Omni inference and required for practical QLoRA fine-tuning.

---

# Data Availability

The original audiovisual data are not redistributed in this repository.

The experiments use the **Seamless Interaction** dataset, specifically the public **Improvised test split**.

The data-preparation notebook documents the download, preprocessing, integrity checks, and participant-centric organisation used in this project.

Derived artifacts such as semantic-summary caches, temporal-feature databases, model checkpoints, and processed audiovisual files may be omitted from the repository because of size or redistribution constraints.

See **[REPRODUCIBILITY.md](REPRODUCIBILITY.md)** for the exact files expected by each stage.

---

# Dissertation Report

The full dissertation report is available here:

**[MSc Dissertation Report](report/Final_Report.pdf)**

The report contains the complete motivation, related work, methodology, experimental protocol, results, discussion, limitations, and future work associated with this repository.

---

# Main Takeaways

1. Direct monolithic audiovisual inference is unreliable for dyadic conversational anomaly detection when speaker attribution and cross-participant reasoning must be performed implicitly.

2. Participant-centric decomposition exposes useful and complementary semantic, temporal, and participation evidence.

3. Evidence availability does **not** guarantee evidence utilisation.

4. Consolidation/output format materially changes classification behaviour even when the underlying evidence is identical.

5. Structured outputs improve auditability but do not establish functionally independent reasoning branches.

6. Controlled ablations reveal strong cross-branch interference, particularly between semantic normality evidence and temporal anomaly interpretation.

7. The best purely predictive development configuration reaches **91.25% accuracy**, but exhibits semantic under-utilisation.

8. Semantic-policy refinement and Setup F1 improve observable semantic fidelity while retaining strong classification performance.

9. Targeted QLoRA further improves Wrong Partner semantic assessment on unseen source conversations.

10. Final source-disjoint evaluation reaches **85.29% overall accuracy**, while explicit Wrong Partner semantic incompatibility doubles from **6/17 to 12/17**.

---

## Citation

If you use or build on this repository, please cite the associated dissertation/report. A formal citation entry can be added here once the final submission metadata is fixed.
