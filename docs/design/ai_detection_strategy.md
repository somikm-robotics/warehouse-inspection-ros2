# AI Detection Strategy

## Overview

The AI detection strategy follows a phased approach, moving from simulation-driven detection in Phase 1 to real-time AI-based perception in Phase 3. This allows the full robotics pipeline — patrol, inspection, alerting, and UI interaction — to be validated before introducing the complexity of AI model training and inference.

Detection is implemented as a pluggable component within the `DefectInspectionNode`. The inspection pipeline does not need to know which detection strategy is active — the `AIDetectionPlugin` and `DbDetectionSimulationPlugin` expose a common interface.

---

## Phase 1 — Simulation-Driven Detection (Current)

### Approach
In Phase 1, the system operates without real-time AI inference. Defect detection is simulated using a defect database to enable full end-to-end system validation while avoiding the time overhead of dataset generation and model training.

### Cursory Inspection
- Detection is driven by a defect database storing predefined defective item locations
- `DbDetectionSimulationPlugin` reads this data and determines whether a defect is present at a given inspection point
- Detection is location-aware and deterministic — not random

### Deep Inspection
- Robot captures images using onboard cameras at deep inspection points
- Images are stored for offline analysis
- No real-time inference is performed at this stage

### Outcome
- Full patrol, inspection, alerting, and UI interaction pipeline validated
- Detection behaviour is realistic — tied to actual defect locations in the database
- No dependency on AI model training for initial system validation

---

## Phase 3 — Real-Time AI-Based Detection (Planned)

### Approach
In Phase 3, simulation-driven detection is replaced with real-time AI-based perception. Camera feeds are processed using trained models to detect and classify defects during patrol.

### Cursory Inspection
- Camera feeds processed using trained AI models in real time
- Detection performed for:
  - structural damage
  - packaging damage
  - visible spillage
  - label and barcode issues

### Deep Inspection
- High-confidence inspection performed using AI inference on captured images
- Detection becomes data-driven, model-based, and adaptive to real-world variations

### AI Pipeline

**Dataset Generation**
- Synthetic datasets generated from a single base product image and a single base packed item image
- Defect variations introduced through controlled image modification (Blender or equivalent):
  - structural damage
  - packaging defects
  - label variations
  - spillage overlays
- Images reused across multiple rack faces, shelf positions, and dispatch items
- Mapping of images to locations driven by inventory data and defect configuration

**Model Training**
- Models trained for detection, classification, and estimation where applicable
- Candidate architectures: YOLO (real-time detection), ResNet variants (classification)

**Image Set (Phase 1 baseline)**

| Image Type | Count |
|-----------|-------|
| Base product image | 1 |
| Base packed item image | 1 |
| `large_structural_damage` | 1 |
| `packaging_damage_visible` | 2 |
| `missing_or_incorrect_labels` | 3 |
| `barcode_or_qr_mismatch` | 2 |
| `misplaced_items` | 2 |
| `visible_spillage_liquid` | 2 |
| Rack model / image | 1 |
| Environmental / obstacle images | 3 |
| **Total** | **~18** |

### Outcome
- Detection fully automated, real-time, and scalable
- System transitions from simulation-driven behaviour to perception-driven autonomy
- `AIDetectionPlugin` replaces `DbDetectionSimulationPlugin` within the inspection pipeline without architectural changes

---

## Plugin Architecture

```
DefectInspectionNode
    ↓
Detection interface
    ├── Phase 1: DbDetectionSimulationPlugin  → database-driven, deterministic
    └── Phase 3: AIDetectionPlugin            → real-time model inference
    ↓
bool detected → MissionHandlerNode
```

The wider system is unaffected by which plugin is active. `DefectInspectionNode` delegates detection and receives a result — the detection strategy is fully encapsulated within the plugin.

---

## Key Design Characteristics

**Phased validation approach**
Full pipeline validated in simulation before introducing AI dependency. Reduces risk and allows parallel development of robotics and AI components.

**Pluggable detection architecture**
Detection strategy is encapsulated within the plugin layer. Switching from simulation-driven to AI-based detection requires no changes to navigation, patrol, or orchestration components.

**Minimal image set with high reuse**
A single base product image and rack model are reused across the entire warehouse. Defects are represented as controlled variations, minimising dataset generation effort while maintaining inspection coverage.

**Clear path from simulation to autonomy**
Phase 1 establishes the inspection pipeline and data capture infrastructure. Phase 3 builds on this foundation, replacing simulated detection with trained model inference.
