
# LineGate AOI
### Multi-View AI Quality Inspection for SMT First-Board Validation

**Superb AI × BDAI Vision AI Hackathon | Team Koala**

## 1. Project Overview

LineGate AOI is a prototype AI-assisted quality inspection system designed for first-board inspection in SMT manufacturing.

The system analyzes PCB images captured from three viewpoints to inspect component placement and solder joint conditions.

Rather than relying solely on model predictions, the system combines inspection results with rule-based quality decisions and human review.

### Inspection Workflow

1. Inspect component placement using the V1 top-view image.
2. Analyze solder joint conditions using V2 and V2.1 angled-view images.
3. Combine inspection results from all three viewpoints.
4. Generate a RELEASE, HOLD, or REVIEW recommendation.
5. Allow an operator to review the evidence and make the final decision.

---

## 2. My Contributions

This project was developed collaboratively by Team Koala.

My main responsibilities included:

### Data Preparation
- Collected and organized PCB inspection images.
- Participated in image labeling and annotation preprocessing.
- Processed annotation data for model training.

### Quality Decision Logic
- Implemented rules for RELEASE, HOLD, and REVIEW decisions.
- Integrated inspection outcomes from multiple viewpoints.
- Supported human review for uncertain or incomplete inspection results.

### Web Application Development
- Developed and integrated the web-based inspection workflow.
- Connected model predictions to inspection result visualization.
- Implemented interfaces for reviewing inspection outcomes.

### Experimental Analysis
- Analyzed model evaluation results.
- Compared inspection performance across different viewpoints.
- Contributed to experimental documentation and reporting.

---

## 3. Dataset and Inspection Tasks

The project used publicly available PCB assembly images from the SolDef_AI dataset.

Three image viewpoints were selected for inspection.

| View | Inspection Target | Task |
|---|---|---|
| V1 | Component placement | Object Detection |
| V2 | Solder joint condition | Instance Segmentation |
| V2.1 | Solder joint condition | Instance Segmentation |

### V1: Component Placement

The V1 model classifies component placement into:

- `good_BBBox`: Acceptable component placement
- `no_good_BBBox`: Abnormal component placement

The dataset contains 268 bounding-box annotations.

### V2 and V2.1: Solder Joint Inspection

The solder inspection model uses four classes:

- `good`: Acceptable solder joint
- `exc_solder`: Excess solder
- `poor_solder`: Insufficient solder
- `spike`: Protruding solder defect

The final dataset contains 260 images and 627 polygon annotations.

---

## 4. AI Models and Experimental Results

Two AI models were used for different inspection tasks.

### Component Placement: RF-DETR Small

The V1 inspection model was trained using Superb AI Platform.

The team initially explored instance segmentation, but later changed the task to object detection to better match the inspection objective.

**Final Test Results**

| Metric | Result |
|---|---|
| Precision | 81.5% |
| Recall | 81.5% |
| F1-score | 81.5% |
| mAP | 0.658 |
| mAP@50 | 0.746 |

Test images: 27

### Solder Joint Inspection: YOLO11s-seg

V2 and V2.1 images were combined to train an instance segmentation model.

**Final Test Results**

| Metric | Result |
|---|---|
| Mask Precision | 0.810 |
| Mask Recall | 0.820 |
| Mask mAP50 | 0.861 |
| Mask mAP50-95 | 0.689 |

Test images: 26

These results were obtained from the project's internal test datasets.

---

## 5. Quality Decision System

The system combines the three inspection results to recommend an action.

| Decision | Condition | Action |
|---|---|---|
| RELEASE | All required inspections indicate normal conditions | Recommend production release |
| HOLD | At least one inspection identifies a defect | Recommend production hold |
| REVIEW | Low-confidence predictions or missing inspection data | Request operator review |

The decision system is intended to support quality engineers rather than replace their judgment.

Operators can review defect locations, model confidence, and inspection evidence before confirming the final action.

---

## 6. System Implementation

The prototype was implemented using Python-based backend and web application tools.

### Technology Stack

- **Python:** Data processing and application logic
- **RF-DETR:** Component placement detection
- **YOLO11s-seg:** Solder joint instance segmentation
- **FastAPI:** Model inference backend
- **Streamlit:** Inspection interface and result visualization

### System Components

- `frontend/app.py`: Inspection interface and result integration
- `backend/main.py`: Solder inspection inference API
- `backend/decision_engine.py`: Decision rules for inspection results
- `models/`: Trained solder inspection model
- `demo_images/`: Example inspection images

The V1 model uses Superb AI Deployment, while the V2/V2.1 model runs through the FastAPI backend.

---

## 7. Limitations and Future Improvements

The project was developed as a proof of concept rather than a production-ready inspection system.

Current limitations include:

- Relatively small evaluation datasets
- Limited validation across different PCB components
- Dependence on Superb AI Deployment for V1 inference
- Remaining false negatives in component placement inspection

Potential improvements include:

- Evaluating the system on additional PCB datasets
- Improving defect detection under different imaging conditions
- Investigating uncertainty-aware inspection decisions
- Validating the system in real manufacturing environments

---

## 8. Project Takeaways

This project provided practical experience in connecting AI-based defect detection with manufacturing quality decisions.

A key lesson was that inspection performance should be considered together with operational decision rules.

Even when an AI model produces a prediction, uncertain results may require additional inspection or human verification.

This motivated my interest in reliable AI-based quality inspection and data-driven manufacturing systems.

---

## 9. Project Information

**Program:** Superb AI × BDAI Vision AI Hackathon

**Team:** Koala

**Project Type:** Team Project

**Application:** SMT First-Board Quality Inspection
