---
title: Rosepetal Research
description: Applied AI research for industrial visual inspection, process understanding, and edge-ready quality control.
---

# Rosepetal Research

**Applied AI research for industrial visual inspection, process understanding, and edge-ready quality control.**

Rosepetal Research is the research department of **Rosepetal AI**, focused on developing next-generation computer vision and multimodal AI systems for manufacturing environments.

Our mission is to bridge the gap between cutting-edge AI research and real industrial deployment: models that are accurate, explainable, data-efficient, and efficient enough to run in production lines, close to the machines, cameras, operators, and quality teams that use them every day.

## Research Mission

Industrial quality control is not a generic computer vision problem.

Manufacturing environments introduce constraints that standard AI systems often fail to handle properly. Rosepetal Research exists to design AI systems that understand these constraints from the beginning.

We work on architectures, datasets, training strategies, evaluation methods, and deployment patterns that make advanced AI usable in real factories.

## Current Research Projects

Rosepetal Research currently maintains six active research projects.

| Project | Focus | Status |
|---|---|---|
| [**Detector-to-Scout Distillation**](#detector-to-scout-distillation) | Distilling box-supervised defect detectors into compact image-level anomaly scouts | <Badge type="tip" text="Public" /> |
| [**Selective Inspection**](#selective-inspection) | Risk-calibrated weak supervision for efficient industrial visual inspection | <Badge type="tip" text="Public" /> |
| [**RP-ForgeVL**](#rp-forgevl) | Few-shot grounded multimodal industrial visual inspection | <Badge type="warning" text="Public release soon" /> |
| [**RP-DETR**](#rp-detr) | Real-time anomaly detection for high-resolution manufacturing inspection | <Badge type="warning" text="Public release soon" /> |
| [**RP-ProcessLens**](#rp-processlens) | Vision-based process verification from manufacturing video | <Badge type="warning" text="Public release soon" /> |
| [**RP-IAD**](#rp-iad) | Large-scale industrial anomaly detection dataset construction | <Badge type="warning" text="Public release soon" /> |

::: info Repository availability
Repositories marked <Badge type="warning" text="Public release soon" /> are currently private and will be published publicly shortly.
:::

## Projects

### Detector-to-Scout Distillation <Badge type="tip" text="Public" />

[**Detector-to-Scout Distillation**](https://github.com/rosepetal-ai/rosepetal-research-detector-to-scout-distillation) is the companion code repository for the paper *"Boxes to Scores: Distilling Box-Supervised Defect Detectors into Compact Image-Level Anomaly Scouts"*.

Industrial inspection lines ultimately need one image-level decision — pass or flag — yet the most accurate defect models are box-supervised detectors: annotation-hungry to train and larger than the deployment slot they must fill. This project implements a recipe for transferring the knowledge inside such a detector **across tasks**: from box-level detection into a compact student that outputs only an image-level anomaly score and runs without its teacher.

The pipeline has three stages:

1. **Target construction**: cached teacher predictions are converted into per-image pseudo-box target records.
2. **Distillation training**: Hungarian matching assigns targets to the student's queries; matched-query losses plus an image-label BCE train a compact multi-scale transformer student ("RT-Scout", 6.81 M parameters).
3. **Teacher-free deployment**: one forward pass, one scalar anomaly score, with ONNX export for edge inference.

#### Key Research Questions

- How can box-level detection knowledge be distilled into an image-level scoring task?
- How small can a student be while preserving the teacher's ranking of defective images?
- How can pseudo-box supervision train useful representations without deployed detection heads?
- How can distilled scouts fit real deployment slots on inline inspection hardware?

#### Research Direction

Detector-to-Scout Distillation targets deployments where a full detector is too heavy for the available compute budget, but its accumulated knowledge should not be discarded. It connects naturally with [Selective Inspection](#selective-inspection), which studies where such compact scouts fit in an inspection cascade.

### Selective Inspection <Badge type="tip" text="Public" />

[**Selective Inspection**](https://github.com/rosepetal-ai/rosepetal-research-selective-inspection) is the companion code repository for the paper *"Learning When Not to Inspect: Risk-Calibrated Weak Supervision for Efficient Industrial Visual Inspection"*.

Most inline production images are normal. This project implements a risk-calibrated selective-inspection cascade — the pieces a production line needs to *spend compute only where suspicion remains*:

1. **A weak scout**: a lightweight image scorer trained from **image-level OK/NOK labels only** — no boxes, no masks — the labels a production line accumulates first.
2. **A risk-calibrated Fast-OK-Exit gate**: an exit threshold calibrated on validation data to keep wrongly-exited NOK images within an operator-chosen missed-NOK budget. Images scoring below it are admitted OK on the spot and bypass all downstream compute.
3. **A split-conformal accept/review/reject decision layer**: thresholds derived from validation OK scores, with a finite-sample guarantee on the OK-rejection rate.

The expensive inspector model is deliberately pluggable: any callable that maps an image to an anomaly score works unchanged. All deployment knobs — exit budget, decision-layer strictness, inspector choice — are post-hoc and reconfigure a deployed line without retraining.

#### Key Research Questions

- How can a production line safely skip inspection for the majority of clearly normal images?
- How can exit thresholds carry explicit, operator-chosen risk budgets?
- How can conformal methods provide finite-sample guarantees on inspection decisions?
- How can cascade behavior be reconfigured post-hoc, without retraining?

#### Research Direction

Selective Inspection addresses the compute economics of inline inspection: expensive models should only run where they add value. Combined with compact scouts from [Detector-to-Scout Distillation](#detector-to-scout-distillation), it defines a full efficiency-oriented inspection cascade.

### RP-ForgeVL <Badge type="warning" text="Public release soon" />

[**RP-ForgeVL**](https://github.com/rosepetal-ai/rosepetal-research-RP-ForgeVL) is a research-oriented project for **few-shot grounded multimodal industrial visual inspection**.

The project explores how inspection models can learn acceptance criteria from a small set of reference examples:

- **OK samples** that define acceptable variation.
- **NOK samples** that describe known defects.
- Localization annotations such as:
  - Bounding boxes.
  - Oriented bounding boxes.
  - Polygons.
  - Segmentation masks.

The goal is to move beyond traditional supervised inspection pipelines that require large quantities of annotated defect data. Instead, RP-ForgeVL investigates models that can reason from examples, visual references, defect descriptions, and grounded annotations.

#### Key Research Questions

- How can a model learn what is acceptable from a small set of OK samples?
- How can textual defect descriptions improve visual inspection?
- How can multimodal models ground defects in industrial images?
- How can few-shot inspection be made reliable enough for production use?
- How can quality teams supervise, correct, and refine model behavior efficiently?

#### Research Direction

RP-ForgeVL is especially relevant for industrial cases where defects are rare, expensive to reproduce, or difficult to annotate exhaustively. It aims to reduce the amount of data needed to deploy new inspections while preserving traceability and human supervision.

### RP-DETR <Badge type="warning" text="Public release soon" />

[**RP-DETR**](https://github.com/rosepetal-ai/rosepetal-research-RP-DETR) is a real-time anomaly detection architecture for manufacturing.

The project focuses on detecting tiny defects in high-resolution industrial images by combining two complementary stages:

1. **Fast full-image scanning**
   The model first processes the complete image using an efficient low-cost pass to identify suspicious regions.

2. **High-resolution focused inspection**
   Only the suspicious regions are then inspected at higher resolution, reducing unnecessary computation while preserving sensitivity to small defects.

RP-DETR also introduces a **normality-verification stage** using OK samples to reduce false positives. This is particularly important in manufacturing, where acceptable variation can be large and false rejects can create significant operational cost.

#### Key Research Questions

- How can we detect very small defects without processing the entire image at maximum resolution?
- How can DETR-style architectures be adapted for industrial anomaly detection?
- How can OK references be used to verify whether a suspicious region is truly defective?
- How can the model remain fast enough for inline inspection?
- How can anomaly detection be scaled to large OK/NOK industrial datasets?

#### Research Direction

RP-DETR targets production lines where high-resolution inspection is required but full-resolution inference is too expensive. The project is designed for inline quality control scenarios where latency, throughput, and false-positive control are critical.

### RP-ProcessLens <Badge type="warning" text="Public release soon" />

[**RP-ProcessLens**](https://github.com/rosepetal-ai/rosepetal-research-RP-ProcessLens) is a vision-based AI system for **quality control of manufacturing processes**.

While many inspection systems focus only on the final product, RP-ProcessLens focuses on the process itself. It analyzes manufacturing video to verify whether operators, tools, machines, and parts follow the expected production procedure.

The system is designed to detect and explain deviations such as:

- Missed steps.
- Wrong tool usage.
- Incorrect operator-machine interaction.
- Abnormal timing.
- Unsafe operations.
- Unexpected process sequences.
- Missing or misplaced components during assembly.

#### Key Research Questions

- How can video models understand manufacturing procedures over time?
- How can an AI system compare observed actions with an expected process?
- How can process deviations be detected early, before they become product defects?
- How can evidence be presented in a way that is useful for quality teams?
- How can process monitoring remain privacy-conscious, traceable, and robust?

#### Research Direction

RP-ProcessLens expands Rosepetal's research scope from product inspection to process intelligence. The goal is to provide quality teams with actionable, timestamped evidence that helps detect issues earlier and improve process reliability.

### RP-IAD <Badge type="warning" text="Public release soon" />

[**RP-IAD**](https://github.com/rosepetal-ai/rosepetal-research-RP-IAD) is a research project focused on the **construction of a large-scale industrial anomaly detection dataset**.

Most public anomaly detection benchmarks are built from a small number of object categories, limited defect variability, and controlled acquisition conditions. RP-IAD aims to build a dataset that better reflects the realities of industrial inspection:

- A wide range of materials, parts, and product categories.
- Diverse defect types, scales, and appearances.
- Realistic acquisition conditions, including varying lighting, angles, and resolutions.
- OK and NOK samples curated to represent acceptable variation and known failure modes.
- Localization annotations suitable for detection, segmentation, and grounded inspection.

The project covers the full dataset lifecycle: acquisition protocols, annotation methodology, quality assurance, structure, splits, and release format.

#### Key Research Questions

- What characteristics should an industrial anomaly detection dataset have to support real deployment, not just academic benchmarking?
- How can OK variability be captured systematically across many product categories?
- How can annotations remain consistent across diverse defect types and inspection contexts?
- How can the dataset support multiple research directions, including few-shot, grounded, and high-resolution inspection?
- How can dataset construction itself be made traceable, reproducible, and extensible?

#### Research Direction

RP-IAD provides the data foundation for the rest of Rosepetal Research. It is designed to feed into [RP-ForgeVL](#rp-forgevl), [RP-DETR](#rp-detr), and future inspection models, ensuring that architectural research is evaluated under conditions that resemble real manufacturing rather than idealized benchmarks.

## Research Themes

Across all projects, Rosepetal Research focuses on several shared technical themes.

### Data-Efficient Learning

Industrial defect data is scarce, imbalanced, and often expensive to annotate. We research methods that reduce the data required to build useful inspection systems, including few-shot learning, reference-based learning, OK-sample modeling, and human-supervised auto-labeling.

### Grounded and Explainable Inspection

Quality teams need to know not only whether a sample is OK or NOK, but also why. Our systems aim to provide grounded evidence: regions, masks, examples, scores, comparisons, and explanations that can be reviewed by humans.

### High-Resolution Visual Understanding

Many manufacturing defects are small, subtle, or local. Rosepetal Research investigates architectures that can reason over large images while preserving detail where it matters.

### Real-Time and Edge Deployment

Industrial AI must work under real production constraints. Research prototypes are designed with deployment in mind: latency, memory, GPU utilization, multi-camera operation, and robustness on edge hardware.

### Human-in-the-Loop Quality Control

Rosepetal Research does not assume that AI replaces quality experts. Instead, we design systems where operators, technicians, and quality engineers can supervise, correct, validate, and improve AI behavior over time.

### Process-Aware AI

Beyond isolated image classification, manufacturing requires understanding sequences, operations, timing, tools, and context. RP-ProcessLens extends our work toward video-based process verification and operational intelligence.

## Research-to-Product Philosophy

Rosepetal Research is applied by design.

A research idea is valuable when it can eventually improve real inspection systems, reduce deployment time, increase reliability, or give quality teams better tools.

Our research workflow follows four principles:

1. **Start from real industrial constraints**
   Every project begins with the realities of production: cameras, lighting, cycle time, edge hardware, operator workflows, and customer data.

2. **Prototype scientifically**
   We validate ideas through controlled experiments, datasets, metrics, and ablation studies.

3. **Design for transfer**
   Architectures and training methods should be compatible with Rosepetal's broader platform, datasets, edge inference stack, and Rosepetal Flows.

4. **Preserve traceability**
   Industrial AI decisions must be auditable. Models should produce outputs that can be reviewed, compared, and improved.
