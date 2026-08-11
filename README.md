# Participant-Centric Evidence Consolidation for Dyadic Conversational Anomaly Detection with Multimodal LLMs

This repository contains the code and experimental notebooks accompanying the MSc dissertation **“Participant-Centric Evidence Consolidation for Dyadic Conversational Anomaly Detection with Multimodal LLMs.”**

The project investigates conversation-level anomaly detection in dyadic audiovisual interactions using **Qwen2.5-Omni-7B**. Three controlled anomaly families are considered: **Wrong Partner**, **Lag Partner**, and **Silent Partner**. The final framework analyses participants independently, constructs structured **semantic, temporal, and participation evidence**, and consolidates these evidence sources for conversation-level classification.

This README acts as a guide to the submitted supporting material. It maps the experiments reported in the dissertation to the corresponding notebooks, explains the order in which the notebooks relate to one another, and lists the data, cached artifacts, model files, and other resources required to reproduce each stage.

## Dissertation Report

The complete methodology, experimental analysis, results, discussion, and conclusions are available in the dissertation report:

**[MSc Dissertation Report](report/Final_Report.pdf)**

> **Note:** The repository contains the experimental code and saved notebook outputs. Large audiovisual data, model checkpoints, and some intermediate cached artifacts are not distributed directly with the repository. The required inputs for each notebook are documented below.

## Experimental Pipeline and Notebook Map

The diagram below shows the experimental progression of the dissertation and maps each major stage to the corresponding notebook(s) in this repository.

```mermaid
flowchart TD

    A["Seamless Interaction Dataset<br/><br/>01_data_preparation/<br/>01_download_and_preprocess_seamless_interaction.ipynb"]

    A --> B["Direct Monolithic Baseline<br/><br/>02_direct_baseline/<br/>01_direct_monolithic_video_only_baseline.ipynb"]

    B --> C["Participant-Centric Redesign"]

    C --> S1["Semantic Evidence<br/><br/>03_isolated_branches/<br/>01_semantic_normal_vs_wrong_partner.ipynb"]

    C --> T1["Temporal Evidence Development<br/><br/>03_isolated_branches/<br/>02_temporal_full_development_pipeline.ipynb"]

    T1 --> T2["Selected Temporal Representation<br/><br/>03_isolated_branches/<br/>03_temporal_selected_representation_reported_isolated_result.ipynb"]

    C --> P1["Participation Evidence Development<br/><br/>03_isolated_branches/<br/>04_participation_branch_development.ipynb"]

    S1 --> DB
    T2 --> DB
    P1 --> DB

    DB["Build Common 400-Case Structured Evidence Database<br/><br/>04_consolidation/<br/>00_build_consolidation_evidence_database.ipynb"]

    DB --> CF{"Consolidation Format Comparison"}

    CF --> C1["Binary Only<br/><br/>04_consolidation/<br/>01_binary_only_consolidation.ipynb"]

    CF --> C2["Structured R1<br/><br/>04_consolidation/<br/>02_structured_r1_consolidation.ipynb"]

    CF --> C3["Free-Form Rationale<br/><br/>04_consolidation/<br/>03_free_form_rationale_consolidation.ipynb"]

    CF --> C4["Structured R1 + Rationale<br/><br/>04_consolidation/<br/>04_structured_r1_with_free_form_rationale_consolidation.ipynb"]

    C2 --> A1["Participation Branch Ablation<br/>Selected: speaks only<br/><br/>05_ablations/participation/<br/>01_participation_branch_ablation.ipynb"]

    A1 --> A2["Local Temporal Ablation<br/>Selected: response offsets only<br/><br/>05_ablations/temporal/<br/>02_local_temporal_branch_ablation.ipynb"]

    A2 --> A3["Global Temporal Ablation<br/>Selected: full global branch<br/><br/>05_ablations/temporal/<br/>03_global_temporal_branch_ablation.ipynb"]

    A3 --> A4["Semantic Branch Ablation<br/>Cross-Branch Interference Identified<br/><br/>05_ablations/semantic/<br/>04_semantic_branch_ablation.ipynb"]

    A4 --> R1["Semantic Policy Refinement<br/><br/>06_setup_f1/<br/>01_semantic_policy_refinement.ipynb"]

    R1 --> F1["Semantic Payload Ablation → Setup F1<br/><br/>06_setup_f1/<br/>02_setup_f1_semantic_payload_ablation.ipynb"]

    F1 --> FT["Targeted Semantic QLoRA Fine-Tuning<br/>+ Adapter-Held-Out Evaluation<br/><br/>07_finetuning/<br/>01_targeted_semantic_qlora_finetuning.ipynb"]

    FT --> U1["Build Final Unseen Source-Disjoint Database<br/>17 Sources → 68 Cases<br/><br/>08_final_source_disjoint_evaluation/<br/>01_build_final_unseen_source_disjoint_database.ipynb"]

    U1 --> U2["Final Frozen F1 vs Fine-Tuned F1 Evaluation<br/><br/>08_final_source_disjoint_evaluation/<br/>02_frozen_vs_finetuned_final_source_disjoint_evaluation.ipynb"]
```

The repository folders are numbered according to this progression. Later sections of this README describe, for each notebook, its role in the dissertation, required inputs, generated outputs, reported results, and dependencies on earlier stages.