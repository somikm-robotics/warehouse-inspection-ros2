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

## System Architecture

The system is structured into four packages: **Warehouse Agent Mission**, 
**Warehouse Agent UI**, **Warehouse Agent Shared Infrastructure**, and 
**Warehouse Agent Communication Interfaces**.

For full architecture detail including component diagrams, class diagrams, 
interaction diagrams, and communication model see 
[System Architecture](docs/architecture/system-architecture.md).


---

## Patrol Control

The system supports runtime operator control via the `MissionUINode` 
at two levels:

- **Patrol-level** — start, stop, enable, or disable the overall patrol
- **Checkpoint-level** — activate or deactivate individual inspection points

Control commands are published as ROS2 topics by `MissionControllerNode` 
and consumed by `MissionHandlerNode` and `PatrolPlugin`. Patrol stop and 
checkpoint deactivation are handled gracefully — the system always 
completes its current task before responding.

See [Patrol Control](docs/design/patrol_control.md) for full detail.

---

## Patrol, Path Planning & Navigation

Patrol management, path planning, and navigation execution are separated 
into three distinct layers coordinated by the `MissionHandlerNode`. 
Nav2 handles all path planning and navigation execution in both phases.

Path planning for the next checkpoint runs concurrently with inspection 
at the current checkpoint, eliminating idle time between patrol points.

See [Patrol, Path Planning & Navigation](docs/design/navigation_path_planning.md) 
for full detail including the phased patrol strategy and execution flow.

---

## Warehouse Zones & Inspection Model

The warehouse covers two zones:

**Finished Goods Storage** — 3 aisles, 8 rack blocks (16 rack faces), 
192 product slots. Robot patrols aisles, inspecting rack faces from 
aisle positions across 10 cursory inspection points.

**Dispatch / Shipping** — open floor area with 2 packing rows and 2 
staging rows (12 items total), covered across 4 cursory inspection points.

The system uses a two-tier inspection model — cursory for broad aisle-based 
coverage, deep triggered on defect detection for precise rack-face and 
shelf-position level investigation.

See [Inspection Model](docs/design/inspection_model.md) for full zone 
layouts, inspection point formats, and inspection behaviour.

---

## Sensors

The system uses a combination of standard Gazebo sensors (Camera, LiDAR, 
Depth Camera, Ultrasonic) and custom simulation plugins (Proximity, 
Barcode Scanner, RFID) across both warehouse zones.

Camera is the central inspection sensor. LiDAR handles navigation safety. 
Barcode and RFID provide identity validation. Depth Camera and Proximity 
are context-dependent support sensors.

See [Sensor Strategy](docs/design/sensor_strategy.md) for full defect 
mapping and zone coverage detail.

---


## Detection Strategy

The system uses a phased detection approach:

- **Phase 1 (current)** — simulation-driven detection via a defect database, 
  enabling full pipeline validation without AI training dependency
- **Phase 2 (planned)** — reinforcement learning for adaptive patrol route 
  planning and checkpoint sequencing, with Nav2 handling navigation execution
- **Phase 3 (planned)** — real-time AI-based detection using trained models 
  (YOLO, ResNet variants), transitioning from simulation-driven to 
  perception-driven autonomy

Detection is implemented as a pluggable component — switching phases requires 
no changes to navigation, patrol, or orchestration layers.

Image generation uses a minimal set (~18 images) built from a single base 
product image with controlled defect variations, reused across the warehouse.

See [AI Detection Strategy](docs/design/ai_detection_strategy.md) and 
[RL Patrol Approach](docs/design/rl_patrol_approach.md) for full detail.

---

## Data Layer

The data layer is lightweight — used primarily for lookup and runtime 
state management, not as a data-driven design.

Static configuration (zones, aisles, racks, shelves, defects) is stored 
in YAML / JSON files. Runtime data (inventory, packing items, inspection 
points with coordinates) is stored in a database. Checkpoint activation 
state is persisted by `PatrolPlugin` during patrol execution.

Full schema detail is covered in [Inspection Model](docs/design/inspection_model.md).

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
