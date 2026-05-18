# Reinforcement Learning Patrol Approach

## Overview

The reinforcement learning (RL) extension is designed as a high-level decision-making layer for adaptive warehouse patrol and inspection planning. It is introduced in Phase 2 as an internal strategy replacement within the `PatrolPlugin`, without changes to the wider system architecture.

The objective of the RL layer is to optimise patrol sequencing, inspection prioritisation, and checkpoint selection based on operational state and inspection history — not to replace deterministic navigation provided by Nav2.

---

## Architectural Role

The RL module operates above the navigation layer and interacts exclusively with the `PatrolPlugin` within the mission execution layer.

### RL Responsibilities
- Selecting the next inspection target or patrol checkpoint
- Prioritising high-risk or overdue inspection areas
- Adapting patrol behaviour based on previous inspection outcomes
- Optimising inspection efficiency and travel cost
- Sequencing inspection workflows based on operational state

### Nav2 Responsibilities (unchanged in Phase 2)
- Path planning between inspection points
- Obstacle avoidance
- Localisation
- Deterministic navigation execution

This separation allows the RL layer to focus on strategic decision-making while preserving stable and reliable navigation behaviour throughout.

---

## Plugin Architecture

The `PatrolPlugin` exposes a common interface for selecting the next patrol or inspection target. The rest of the system — including `MissionHandlerNode`, `PathPlanningPlugin`, and `NavigationPlugin` — does not need to know whether the next target was selected from a fixed database schedule or an RL-based policy.

```
MissionHandlerNode
    ↓ requests next target
PatrolPlugin (common interface)
    ├── Phase 1: DbPatrolPlugin  → database-driven ordering
    └── Phase 2: RLPatrolPlugin  → RL policy-driven ordering
    ↓ returns next inspection target
MissionHandlerNode
    ↓
PathPlanningPlugin → NavigationPlugin → Nav2
```

This design allows RL to be introduced without modifying the wider architecture. The Mission Handler continues to call the Patrol Plugin in the same way — the plugin internally switches from a database-driven strategy to an RL-assisted strategy.

---

## Phase 1 vs Phase 2 Patrol Strategy

| | Phase 1 — DbPatrolPlugin | Phase 2 — RLPatrolPlugin |
|--|--------------------------|--------------------------|
| Checkpoint ordering | Predefined database configuration | RL policy |
| Behaviour | Deterministic, structured | Adaptive, optimised |
| Nav2 role | Path planning + navigation execution | Path planning + navigation execution (unchanged) |
| Use case | Initial validation, baseline patrol | Operational efficiency, risk-based prioritisation |

---

## RL Agent Design

### State Space
The RL agent evaluates the following inputs when determining the next patrol target:

- Current robot location
- Historical inspection results
- Defect occurrence frequency per zone / checkpoint
- Zone priority
- Travel distance and cost
- Inspection recency (time since last inspection per point)
- Operational constraints

### Action Space
Based on the evaluated state, the RL agent determines:

- The next patrol checkpoint or inspection target
- Inspection sequencing decisions
- Inspection priority adjustments

### Reward Design
The reward model encourages:

- Efficient patrol coverage
- Prioritisation of high-risk zones
- Timely inspection of overdue areas
- Successful defect detection
- Reduced unnecessary movement

And penalises:

- Excessive travel distance
- Repeated low-value inspections
- Inefficient patrol loops
- Missed inspection priorities

---

## Planned Evolution

The RL architecture is intentionally phased to support progressive capability growth. Future extensions may include:

- Adaptive inspection frequency optimisation
- AI model selection based on product or defect type
- Dynamic resource allocation across inspection workflows
- Multi-robot patrol coordination
- RL-assisted mission orchestration

The overall design goal is a scalable autonomy framework where reinforcement learning enhances operational decision-making while maintaining deterministic and reliable low-level robotic control through Nav2 and ROS2.
