# Autonomous Warehouse Inspection & Monitoring System
**ROS2 · Nav2 · Gazebo · C++ · Python · Docker**

> **Note:** This repository contains system architecture, design documentation, and technical decisions only.
> Code is proprietary to the client. Architecture documentation is available here in full.

---

## Overview

A ROS2-based autonomous inspection and monitoring system for a manufacturing warehouse environment. The robot autonomously patrols a defined warehouse floor, navigating between active checkpoints, inspecting each checkpoint for defects or hazards, and sending structured alerts to an operator UI when issues are detected.

The goal is not simply to move a robot around a warehouse, but to create an **intelligent patrol system** that can monitor operational safety, product integrity, and environmental hazards in a structured and configurable way.

The system is developed in Gazebo simulation with a **simulation-to-real deployment architecture** — designed from the outset to transfer to a physical robot platform.

---

## Core Capabilities

- Autonomous patrol between configurable inspection checkpoints
- Multi-sensor defect and hazard detection across warehouse zones
- Structured alerting to operator UI with defect classification and evidence
- Runtime control — patrol start/stop, checkpoint activation/deactivation
- Phased AI detection strategy — simulation-driven (Phase 1) to real-time AI inference (Phase 2)
- Pluggable sensor and detection architecture

---

## Warehouse Zones

### Finished Goods Storage
- 3 aisles · 8 rack blocks (double-sided) · 16 rack faces
- 4 shelves per rack face · 3 product positions per shelf → **192 product slots**
- Robot moves along aisles, using cross-aisles to switch between them
- Racks inspected from aisle positions

### Dispatch / Shipping Zone
- Open warehouse area (no racks)
- 2 packing rows (P1–P6) · 2 staging rows (S1–S6) → **12 items total**
- 2 aisles (packing + staging) with open movement areas at row ends

---

## Inspection Model

### Cursory Inspection
Broad, aisle-based coverage during normal patrol. Robot slows or pauses at each inspection point.

| Zone | Inspection Points |
|------|------------------|
| Finished Goods Storage | 10 points (2 per aisle × 3 aisles + 4 outer rack face points) |
| Dispatch / Shipping | 4 points (2 aisle-based + 2 end-side) |

### Deep Inspection
Triggered when a potential issue is identified during cursory inspection. Robot moves to a precise rack-face / shelf-position level for detailed verification.

**Inspection Point Format:**
- Cursory: `ZoneId · AisleId · Position (Top/Middle)`
- Deep: `ZoneId · AisleId · RackId · FaceId · ShelfId`

### Inspection Behaviour
- Cursory and deep inspection run **in parallel** with path planning to the next checkpoint
- Upon defect detection at cursory level → robot navigates to corresponding deep inspection point
- After deep inspection → patrol resumes

---

## Defect & Sensor Mapping

| Defect Type | Sensors Used |
|-------------|-------------|
| Obstacles in path | LiDAR, Depth Camera, Proximity (Nav2 primary handler) |
| Large structural damage | Camera, Depth Camera |
| Packaging damage | Camera |
| Missing / incorrect labels | Camera, Barcode Scanner |
| Barcode / QR mismatch | Barcode Scanner, RFID |
| Misplaced items | Camera, RFID |
| Visible spillage | Camera |

### Sensors by Zone

| Sensor | Storage | Dispatch |
|--------|---------|----------|
| Camera | ✔️ | ✔️ |
| LiDAR | ✔️ | ✔️ |
| Depth Camera | ✔️ | optional |
| Ultrasonic / Proximity | ✔️ | optional |
| Barcode Scanner | ✔️ | ✔️ |
| RFID | ✔️ | optional |

**Standard sensors** (URDF / Gazebo Fortress): Camera (dual side-mounted), LiDAR, Depth Camera, Ultrasonic

**Extended sensors** (custom plugins): Proximity (ray-based), Barcode Scanner, RFID

---

## System Architecture

### Component Overview

The system is structured into four packages:

**Warehouse Agent Mission**
- `MissionHandlerNode` — mission workflow sequencing, checkpoint lifecycle
  - `PatrolPlugin` — patrol behaviour and checkpoint traversal
  - `PathPlanningPlugin` — Nav2 path computation between inspection points
  - `NavigationPlugin` — Nav2 navigation execution
- `DefectInspectionNode` — sensor aggregation and defect determination
  - `AIDetectionPlugin` — AI-based detection and classification pipeline
  - `DbDetectionSimulationPlugin` — Phase 1 simulation-driven detection

**Warehouse Agent UI**
- `MissionControllerNode` — patrol enable/disable, checkpoint activation
  - `MissionUINode` — operator control interface (Tinkr)
  - `AlertReceiverNode` — receives and displays structured defect alerts

**Warehouse Agent Shared Infrastructure**
- `ProximityPlugin` — ray-based proximity sensing with threshold logic
- `BarcodeScannerPlugin` — barcode / QR code scanning simulation
- `RfidScannerPlugin` — RFID identity validation simulation

**Warehouse Agent Communication Interfaces**
- Shared ROS2 message and service definitions across packages

### Diagrams

#### Component Diagram
![Component Diagram](docs/diagrams/Component_Diagram.png)

#### Class Diagram
![Class Diagram](docs/diagrams/Class_Diagrams.png)

#### Interaction Diagram — Initial Control (Start/Stop Patrol, Checkpoint Activation)
![Initial Interaction Diagram](docs/diagrams/Initial_Interaction_Diagram.png)

#### Interaction Diagram — Start Patrol Sequence
![Start Patrol Interaction](docs/diagrams/Interaction_Diagram-Start_Patrol.png)

---

## Navigation & Path Planning Approach

The system uses an **incremental, goal-driven navigation model** with overlapping planning and inspection:

1. Each inspection point is treated as a discrete Nav2 navigation goal
2. Robot plans and navigates to the current inspection point
3. On arrival, inspection begins
4. **While inspection is in progress**, the system computes the path to the next inspection point in a separate planning thread
5. On inspection completion, navigation to the next point begins immediately — no delay

This ensures efficient utilisation of time while maintaining clear separation between navigation, planning, and inspection logic.

---

## AI Detection Strategy — Phased Approach

### Phase 1 — Simulation-Driven Detection (Current)
- Cursory inspection driven by a **defect database** storing predefined defective item locations
- `DbDetectionSimulationPlugin` reads location data and determines defect presence deterministically
- Deep inspection: robot captures images and stores for offline analysis — no real-time inference
- Full robotics pipeline (patrol → inspection → alerting → UI) validated without AI training dependency

### Phase 2 — Reinforcement Learning Navigation (Planned)
- RL applied to **navigation route planning** — determining the optimal order and selection of inspection checkpoints based on operational priorities and defect history
- Nav2 retained for actual path planning and navigation execution between points — RL governs checkpoint sequencing only
- Enables adaptive patrol routing that responds to changing warehouse conditions

### Phase 3 — Real-Time AI-Based Detection (Planned)
- Camera feeds processed using trained AI models (YOLO, ResNet variants)
- Real-time detection for structural damage, packaging damage, and visible spillage
- Synthetic datasets generated from base product images for model training
- Detection becomes fully automated, real-time, and scalable
- System transitions from simulation-driven behaviour to perception-driven autonomy

---

## Image Generation Strategy

Designed to minimise effort while supporting full inspection coverage:

| Image Type | Count |
|-----------|-------|
| Base product image (reused across all slots) | 1 |
| Base packed item image | 1 |
| Defect images (damage, labels, spillage, misplacement) | ~12 |
| Rack model / image | 1 |
| Environmental / obstacle images | 3 |
| **Total** | **~18** |

Defect images are generated as controlled variations of the base product image (structural damage, packaging defects, label variations, spillage overlays). Image-to-location mapping is driven by inventory data and defect configuration files.

---

## Data Layer

| Store | Contents |
|-------|----------|
| YAML / JSON | Zone, Aisle, Rack, Shelf definitions; Defect configuration |
| Database | Inventory, PackingItems, InspectionPoints (with coordinates and angles) |

**Inspection Point schema:** `InspectionPointId · ZoneId · AisleId · RackId · RackFace · InspectionType (cursory/deep) · Position · X · Y · Z · Angle`

---

## Tech Stack

| Category | Technologies |
|----------|-------------|
| Robotics Framework | ROS2, Nav2 |
| Simulation | Gazebo Fortress |
| Languages | C++, Python |
| Containerisation | Docker |
| UI | Tinkr |
| Sensing | LiDAR, Camera, Depth Camera, Barcode, RFID, Proximity, Ultrasonic |

---

## Project Status

**Phase 1 — Active Development**
Simulation-driven patrol, inspection, and alerting pipeline in progress.

**Phase 2 — Planned**
Reinforcement learning for adaptive navigation route planning — RL determines checkpoint sequencing, Nav2 executes navigation.

**Phase 3 — Planned**
Real-time AI-based detection.

---

*Contract project — Makdy Techcorp Pvt Ltd (India). Code is proprietary to the client.*
