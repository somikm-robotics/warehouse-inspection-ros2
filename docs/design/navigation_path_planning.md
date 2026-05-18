# Patrol, Path Planning & Navigation

## Overview

The system separates patrol management, path planning, and navigation execution into three distinct layers. Each layer has a clearly defined responsibility and communicates with the others through the `MissionHandlerNode`. Nav2 handles all path planning and navigation execution in both Phase 1 and Phase 2.

---

## 1. Patrol Management

### Responsibility
The `PatrolPlugin` is responsible for maintaining the ordered list of active inspection points and providing the next patrol target to the `MissionHandlerNode`. It does not handle path planning or navigation.

### Inspection Point Management
- Maintains active cursory and deep inspection points
- Provides next inspection target on request
- Reflects runtime checkpoint activation and deactivation
- Verifies checkpoint is still active before returning it as the next target

### Patrol Lifecycle
- Patrol is initiated by `MissionHandlerNode` on receiving a start command
- `PatrolPlugin` sets patrol state to active and returns the ordered inspection point list
- Patrol continues until all points are completed or a stop command is received
- On stop: current task completes gracefully, no further checkpoints are visited

### Phased Patrol Strategy

**Phase 1 — Database-Driven (`DbPatrolPlugin`)**
- Checkpoint ordering retrieved from warehouse database configuration
- Deterministic, structured, and explainable patrol sequences
- Suitable for initial system validation and baseline patrol behaviour

**Phase 2 — RL-Driven (`RLPatrolPlugin`)**
- Checkpoint sequencing determined by a reinforcement learning policy
- RL evaluates current robot location, inspection history, defect frequency, zone priority, travel cost, and inspection recency
- Determines the next patrol target dynamically based on operational state
- Nav2 continues to handle all path planning and navigation execution — RL only influences checkpoint selection and sequencing

See [RL Patrol Approach](rl_patrol_approach.md) for full Phase 2 detail.

---

## 2. Path Planning

### Responsibility
The `PathPlanningPlugin` is responsible for computing paths between inspection points using Nav2. It receives a destination from `MissionHandlerNode` and returns a set of waypoints for navigation execution.

### Path Planning Approach
- Each inspection point is treated as a discrete navigation goal
- `PathPlanningPlugin` calls Nav2 to compute the path and returns waypoints to `MissionHandlerNode`
- Path planning for the next inspection point begins concurrently while inspection is in progress at the current point — eliminating idle time between checkpoints

### Concurrent Planning
```
Robot arrives at inspection point
    ↓
Inspection begins
    ├── Inspection thread: detect defect / no defect
    └── Planning thread: compute path to next active inspection point
    ↓
Inspection completes → navigation begins immediately
```

---

## 3. Navigation Execution

### Responsibility
The `NavigationPlugin` is responsible for executing robot movement to inspection points using Nav2. It receives waypoints from `MissionHandlerNode` and reports completion on arrival.

### Navigation Stack
All navigation is handled by **Nav2**, providing:
- Waypoint-based movement execution
- Obstacle avoidance and recovery behaviours
- Localisation during patrol
- Deterministic navigation in both Phase 1 and Phase 2

### Runtime Interruptions
- Navigation can be stopped asynchronously via patrol stop command
- `NavigationPlugin` completes current movement gracefully before stopping
- Does not halt mid-navigation on stop command

---

## Full Patrol Execution Flow

```
MissionHandlerNode initiates patrol
    ↓
PatrolPlugin returns active cursory + deep inspection points
    ↓
PathPlanningPlugin computes path to first inspection point
    ↓
NavigationPlugin executes movement → arrival confirmed
    ↓
DefectInspectionNode begins inspection
    ↓ (concurrent)
    ├── Inspection: detect defect / no defect
    └── Planning: compute path to next active inspection point
    ↓
┌── No defect detected ──────────────────────────────────────┐
│   NavigationPlugin executes movement to next point         │
│   Process repeats until patrol complete or stopped         │
└────────────────────────────────────────────────────────────┘
    ↓
┌── Defect detected ─────────────────────────────────────────┐
│   Check if corresponding deep inspection point is active   │
│       ↓                                                    │
│   PathPlanningPlugin computes path to deep point           │
│       ↓                                                    │
│   NavigationPlugin executes movement to deep point         │
│       ↓                                                    │
│   DefectInspectionNode performs detailed inspection        │
│       ↓                                                    │
│   Resume patrol → return to main loop                      │
└────────────────────────────────────────────────────────────┘
```

---

## Key Design Characteristics

**Clear separation of responsibilities**
Patrol management, path planning, and navigation execution are fully decoupled. Each layer communicates through `MissionHandlerNode` and can evolve independently.

**Nav2 as stable execution layer**
Nav2 handles all deterministic navigation in both phases. Neither RL checkpoint selection nor inspection logic affects navigation execution behaviour.

**Concurrent planning and inspection**
Path planning for the next checkpoint overlaps with inspection at the current checkpoint, maximising patrol efficiency without tight coupling between components.

**Deep inspection always includes path planning**
On defect detection, the system checks deep point availability, plans a path, then navigates — no direct jump to navigation without a computed path.

**Dynamic runtime adaptability**
Checkpoint activation state is verified before each navigation step. Patrol adapts to runtime configuration changes without requiring restart.
