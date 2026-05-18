# Inspection Model

## Overview

The inspection model defines how the robot covers the warehouse, what it inspects at each location, and how it escalates from broad patrol coverage to precise defect investigation. The model is structured around two inspection tiers — cursory and deep — applied differently across the two warehouse zones.

---

## Warehouse Zones

- **Finished Goods Storage** — rack-based storage with structured aisle movement
- **Dispatch / Shipping Zone** — open floor area with packing and staging rows

### Full Warehouse Layout

```
------------------------------------------------------------------------
| Finished Goods Storage                                               |
|                                                                      |
|  Rack  Aisle 1  Rack    Aisle 2  Rack    Aisle 3  Rack               |
|  R1   |       | R2   |         | R3   |         | R4   |             |
|       |       |      |         |      |         |      |             |
|-------+-------+------+---------+------+---------+------+-------------|
|                                                                      |
|                    Cross-Aisle (Middle)                              |
|                                                                      |
|-------+-------+------+---------+------+---------+------+-------------|
|       |       |      |         |      |         |      |             |
|  R5   |       | R6   |         | R7   |         | R8   |             |
|  Rack  Aisle 1  Rack    Aisle 2  Rack    Aisle 3  Rack               |
|                                                                      |
|                  Cross-Aisle (Front / Entry)                         |
|                      (Connection Area)                               |
|----------------------------|    |------------------------------------|
                             |    |
                             |    |
------------------------------------------------------------------------
| Dispatch / Shipping Zone                                             |
|                                                                      |
|    [Packing Row 1]      Aisle 1      [Packing Row 2]                 |
|                                                                      |
|    [Staging Row 1]      Aisle 2      [Staging Row 2]                 |
|                                                                      |
|                     Open Movement Area                               |
|                                                                      |
------------------------------------------------------------------------
```

---

## Zone 1 — Finished Goods Storage

### Physical Structure

- 3 aisles (primary robot movement paths)
- 8 rack blocks (R1–R8), double-sided → 16 rack faces
- 2 cross-aisles (front entry and middle) for aisle transitions
- Each rack face: 4 shelves × 3 product positions = **12 product slots per face**
- Total: 16 rack faces × 12 = **192 product slots**

### Robot Movement

- Robot moves along aisles
- Cross-aisles used to transition between aisles
- Racks are inspected from aisle positions — robot does not enter rack faces

### Cursory Inspection Points

Inspection is carried out from aisle-based positions providing broad visual coverage of rack faces during normal patrol.

| Location | Points |
|----------|--------|
| Per aisle (top + middle) × 3 aisles | 6 |
| First outer rack face (top + middle) | 2 |
| Last outer rack face (top + middle) | 2 |
| **Total cursory inspection points** | **10** |

**Inspection Point Format:**
```
ZoneId  AisleId  Top
ZoneId  AisleId  Middle
ZoneId  AisleId  Start Top
ZoneId  AisleId  Start Middle
ZoneId  AisleId  End Top
ZoneId  AisleId  End Middle
```

**Coverage from aisle-based points:**
- Both rack faces bordering the aisle
- Multiple shelves and product positions
- Suitable for: packaging damage, label visibility, misplaced items, general visual anomalies

**Coverage from outer rack face points:**
- Outer-facing rack sides not fully visible from aisles
- Ensures complete rack coverage across the zone

### Deep Inspection Points

Triggered when a potential issue is identified during cursory inspection. Enables the robot to move closer to a specific rack face and inspect a specific product position in detail.

**Inspection Point Format:**
```
ZoneId  AisleId  RackId  FaceId  ShelfId
```

**Coverage from deep inspection points:**
- Individual rack face
- Specific shelf position
- Used for: detailed verification, close-range inspection

### Inspection Behaviour

1. Robot moves along aisles performing continuous scanning of rack faces
2. At each cursory inspection point — robot slows or pauses, performs broad inspection
3. If defect detected → robot navigates to corresponding deep inspection point
4. Deep inspection performed at rack-face / shelf-position level
5. Robot resumes patrol after inspection

---

## Zone 2 — Dispatch / Shipping

### Physical Structure

- Open warehouse area — no racks or shelves
- 4 rows total:
  - Packing Row 1 → P1, P2, P3
  - Packing Row 2 → P4, P5, P6
  - Staging Row 1 → S1, S2, S3
  - Staging Row 2 → S4, S5, S6
- **Total: 12 items**

```
Packing Row (Top View)
|--------------------------------|
| P1     | P2     | P3           |
| (Pack) | (Pack) | (Pack)       |
|--------------------------------|

Staging Row (Top View)
|--------------------------------|
| S1      | S2      | S3         |
| (Stage) | (Stage) | (Stage)    |
|--------------------------------|
```

### Robot Movement

- Aisle 1 (Packing Aisle) — between Packing Row 1 and Packing Row 2
- Aisle 2 (Staging Aisle) — between Staging Row 1 and Staging Row 2
- Open movement area at row ends allows turning and repositioning

### Cursory Inspection Points

Inspection is lane-based and edge-based — not item-based.

| Type | Points |
|------|--------|
| Aisle 1 — covers both packing rows | 1 |
| Aisle 2 — covers both staging rows | 1 |
| Front end of rows | 1 |
| Rear end of rows | 1 |
| **Total cursory inspection points** | **4** |

**Inspection Point Format:**
```
ZoneId  AisleId
```

**Coverage from aisles:**
- Packaging condition, labels, misplaced items

**Coverage from ends:**
- Edge-facing items, partial occlusions, items not fully visible from aisles

### Deep Inspection

Not defined for Dispatch zone. All inspection is handled at cursory level.

### Inspection Behaviour

1. Robot moves through aisles
2. Stops at aisle inspection points and end inspection points
3. Performs inspection using camera, barcode scanner, and RFID
4. No escalation to deep inspection

---

## Inspection Summary

| Zone | Cursory Points | Deep Points | Deep Inspection |
|------|---------------|-------------|-----------------|
| Finished Goods Storage | 10 | Per rack face / shelf position | Yes — triggered on defect |
| Dispatch / Shipping | 4 | None | No |
| **Total** | **14** | — | — |


