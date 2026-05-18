# Patrol Control

## Overview

The patrol control system allows an operator to manage warehouse inspection at runtime through the `MissionUINode`. Control commands are published as ROS2 topics by `MissionControllerNode` and consumed by `MissionHandlerNode` and `PatrolPlugin` without requiring system restart.

Two levels of control are supported:

- **Patrol-level control** — start, stop, enable, disable the overall patrol
- **Checkpoint-level control** — activate or deactivate individual inspection points

---

## Control Architecture

```
Operator (MissionUINode)
    ↓
MissionControllerNode
    ↓ publishes ROS2 topics
    ├── enablePatrol (bool)        → MissionHandlerNode
    └── activateCheckpoint (bool)  → MissionHandlerNode
                                   → PatrolPlugin
```

`MissionControllerNode` publishes control commands. `MissionHandlerNode` maintains overall mission state. `PatrolPlugin` maintains checkpoint activation state and persists changes.

---

## Patrol-Level Control

### Enable Patrol
- Operator enables patrol via `MissionUINode`
- `MissionControllerNode` publishes `enablePatrol: true`
- `MissionHandlerNode` receives command and initiates patrol mission
- `PatrolPlugin` sets patrol state to active and begins returning inspection targets

### Disable / Stop Patrol
- Operator disables patrol via `MissionUINode`
- `MissionControllerNode` publishes `enablePatrol: false`
- `MissionHandlerNode` receives command and initiates graceful stop
- Robot completes its current task — does not halt mid-navigation
- No further checkpoints are visited after current task completes
- `PatrolPlugin` sets patrol state to inactive

### Behaviour on Stop
- If navigating → completes current navigation, does not proceed to next checkpoint
- If inspecting → completes current inspection, does not navigate to next checkpoint
- If planning → planning thread terminates, result discarded

---

## Checkpoint-Level Control

### Activate Checkpoint
- Operator activates a checkpoint via `MissionUINode`
- `MissionControllerNode` publishes `activateCheckpoint: true` with checkpoint identifier
- `MissionHandlerNode` updates local mission state (if patrol is active)
- `PatrolPlugin` sets inspection point to active and persists state to database

### Deactivate Checkpoint
- Operator deactivates a checkpoint via `MissionUINode`
- `MissionControllerNode` publishes `activateCheckpoint: false` with checkpoint identifier
- `MissionHandlerNode` updates local mission state (if patrol is active)
- `PatrolPlugin` sets inspection point to inactive and persists state to database
- If robot is currently navigating to the deactivated checkpoint → navigation completes, checkpoint inspection is skipped, patrol moves to next active checkpoint

---

## ROS2 Communication

| Topic | Type | Publisher | Subscriber(s) |
|-------|------|-----------|--------------|
| `enablePatrol` | `bool enable` | `MissionControllerNode` | `MissionHandlerNode` |
| `activateCheckpoint` | `bool activate` | `MissionControllerNode` | `MissionHandlerNode`, `PatrolPlugin` |

Both topics are asynchronous and event-driven. Agents react to published commands rather than being polled.

---

## Runtime Behaviour Summary

| Operator Action | Immediate Effect | Graceful Completion |
|----------------|-----------------|---------------------|
| Enable patrol | Patrol initiates | — |
| Disable patrol | Stop signal published | Current task completes before stopping |
| Activate checkpoint | Checkpoint added to active list | Available for next patrol cycle |
| Deactivate checkpoint | Checkpoint removed from active list | Current navigation to it completes, inspection skipped |

---

## Key Design Characteristics

**Event-driven control**
All control commands propagate via ROS2 pub/sub. Components react to events independently — no tight coupling between UI and execution layers.

**Graceful stop behaviour**
Patrol and checkpoint deactivation never interrupt mid-task execution. The system always completes its current atomic operation before responding to a stop command.

**Runtime adaptability**
Checkpoint activation state can change during active patrol. The system reflects changes at the next checkpoint selection step without requiring restart.

**Persistent checkpoint state**
Checkpoint activation and deactivation is persisted to the database by `PatrolPlugin`, ensuring state is maintained across patrol cycles.
