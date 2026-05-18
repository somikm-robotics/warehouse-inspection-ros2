# Sensor Strategy

## Overview

The sensor strategy defines which sensors are used across the warehouse, how they map to defect types, and how they are integrated into the simulation environment. The approach uses a combination of standard Gazebo sensors and custom simulation plugins to cover all required inspection modalities.

---

## Sensor Configuration

### Standard Sensors (URDF / Gazebo Fortress)
Integrated directly via URDF and Gazebo Fortress:

- **Camera** (dual side-mounted) — primary inspection sensor across both zones
- **LiDAR** — navigation safety and obstacle detection
- **Depth Camera** — structural damage and volume / shape anomaly detection
- **Ultrasonic** — close-range obstacle detection in tight aisle environments

### Extended Sensors (Custom Plugins)
Where Gazebo does not provide native support, custom plugins extend the simulation:

- **ProximityPlugin** — ray-based proximity sensing with threshold logic
- **BarcodeScannerPlugin** — barcode / QR code scanning simulation
- **RfidScannerPlugin** — RFID identity validation simulation

---

## Defect & Sensor Mapping

| Defect Type | Sensors Used | Notes |
|-------------|-------------|-------|
| `obstacles_in_path` | LiDAR, Depth Camera, Ultrasonic, Proximity | Handled primarily by Nav2 |
| `large_structural_damage` | Camera, Depth Camera | Visual + depth-based detection |
| `packaging_damage_visible` | Camera | Visual inspection |
| `missing_or_incorrect_labels` | Camera, Barcode Scanner | Visual + barcode validation |
| `barcode_or_qr_mismatch` | Barcode Scanner, RFID | Identity / mismatch detection |
| `misplaced_items` | Camera, RFID | Location + visual mismatch |
| `visible_spillage_liquid` | Camera | Visual anomaly detection |

---

## Sensor Coverage by Zone

| Sensor | Finished Goods Storage | Dispatch / Shipping |
|--------|----------------------|--------------------|
| Camera | ✔️ | ✔️ |
| LiDAR | ✔️ | ✔️ |
| Depth Camera | ✔️ | optional |
| Ultrasonic / Proximity | ✔️ | optional |
| Barcode Scanner | ✔️ | ✔️ |
| RFID | ✔️ | optional |

**Camera** — central sensor used across both zones for visual inspection.
**LiDAR** — navigation safety and obstacle detection.
**Barcode / RFID** — identity validation and misplacement detection.
**Depth Camera / Proximity** — support sensors, context-dependent.

---

## Sensor Role Summary

| Sensor | Primary Role |
|--------|-------------|
| Camera | Packaging damage, label issues, spillage, misplaced items |
| LiDAR | Navigation, obstacle detection, aisle safety |
| Depth Camera | Structural damage, volume and shape anomalies |
| Ultrasonic / Proximity | Close-range obstacle detection in constrained spaces |
| Barcode Scanner | Label validation, barcode / QR mismatch detection |
| RFID | Item identity validation, misplacement detection |
