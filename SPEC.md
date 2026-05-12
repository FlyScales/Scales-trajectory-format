# Scales Trajectory Format — Specification v1.0

**Status:** Draft  
**Date:** May 2026  
**Authors:** Amit Zadok, Scales  
**License:** CC BY 4.0

---

## 1. Overview

The Scales Trajectory Format (STF) is a time-series data format for recording 
body-flight performance inside wind tunnels. It is designed to be:

- **Simple:** Plain JSONL — one JSON object per line, no binary encoding
- **Open:** No vendor lock-in. Any language can read or write STF.
- **Extensible:** New device types and fields can be added without breaking 
  existing parsers
- **Privacy-aware:** Personal identity is never stored inside a flight file

---

## 2. File Format

An STF file is a UTF-8 encoded JSONL file (`.jsonl` extension).  
Each line is a valid, self-contained JSON object.  
Lines must not be empty except at end of file.

### 2.1 Line types

| Line | Type field | Description |
|------|-----------|-------------|
| First line | `"_header": true` | File metadata |
| All subsequent lines | *(absent)* | Flight frames |

---

## 3. Header Object

The first line of every STF file must be a header object.

### Schema

```json
{
  "_header": true,
  "schema_version": "1.0",
  "flight_id": "2026-04-19_flybox_il_f01",
  "frame_format": "v1",
  "tunnel_id": "flybox_il",
  "pilot_id": "amit_zadok",
  "coordinate_frame": "tunnel",
  "units": "meters",
  "created_by": "scales-server@0.1.0"
}
```

### Field definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `_header` | bool | ✅ | Must be `true` |
| `schema_version` | string | ✅ | STF version, e.g. `"1.0"` |
| `flight_id` | string | ✅ | Unique flight identifier. Convention: `YYYY-MM-DD_tunnel_index` |
| `frame_format` | string | ✅ | Frame schema version. Currently `"v1"` |
| `tunnel_id` | string | ✅ | References tunnel registry |
| `pilot_id` | string | ✅ | References pilot registry (never contains personal data inline) |
| `coordinate_frame` | string | ✅ | Coordinate system. Currently `"tunnel"` |
| `units` | string | ✅ | Distance units. Currently `"meters"` |
| `created_by` | string | ❌ | Software that wrote the file |

---

## 4. Frame Object

Every line after the header is a frame — a snapshot of all tracked devices 
at a single point in time.

### Schema

```json
{
  "t": 1775400745.430,
  "devices": {
    "tag_hip": {
      "position": {"x": 1.76, "y": 1.49, "z": 1.94},
      "orientation": {"qw": 0.998, "qx": 0.012, "qy": -0.045, "qz": 0.038},
      "quality": "good",
      "raw_distances": {"A1": 1.23, "A2": 2.10, "A3": 1.87, "A4": 2.45}
    }
  }
}
```

### Field definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `t` | float | ✅ | Unix timestamp in seconds (UTC) |
| `devices` | object | ✅ | Map of device_id → device state |
| `devices.{id}.position` | object | ✅ | x, y, z in tunnel coordinate frame (meters) |
| `devices.{id}.orientation` | object | ❌ | Unit quaternion: qw, qx, qy, qz |
| `devices.{id}.quality` | string | ✅ | `"good"` \| `"interpolated"` \| `"dead_reckoning"` \| `"dropped"` |
| `devices.{id}.raw_distances` | object | ❌ | Raw UWB anchor distances (meters) |

---

## 5. Quality Flags

| Value | Meaning |
|-------|---------|
| `good` | Full UWB fix, all anchors responding |
| `interpolated` | Partial anchor data, position estimated |
| `dead_reckoning` | No UWB data, position from IMU only |
| `dropped` | Frame could not be computed |

---

## 6. Coordinate Frame

The `"tunnel"` coordinate frame is defined from the perspective of a flyer 
facing the tunnel operator window:

- **Origin:** center of the tunnel flight chamber floor
- **X axis:** pointing toward the tunnel operator window (forward)
- **Y axis:** pointing left (from flyer perspective, facing operator)
- **Z axis:** pointing up

Tunnel geometry (dimensions, anchor positions) is stored in the tunnel 
registry, not inside the STF file.

---

## 7. Derived Fields

The following fields are intentionally **not stored** in STF files. 
They must be computed by the reader:

| Field | How to compute |
|-------|---------------|
| `velocity` | Centred finite differences over position frames |
| `speed` | `sqrt(vx² + vy²)` for horizontal, `vz` for vertical |
| `pitch / roll / yaw` | Derived from orientation quaternion |
| `maneuver labels` | Application-level, not part of STF |

This keeps raw files small, immutable, and free of implementation assumptions.

---

## 8. Versioning

STF uses semantic versioning (`major.minor`).

- **Minor version bump:** new optional fields added. Backward compatible.
- **Major version bump:** breaking changes to required fields or structure.

Parsers must check `schema_version` and reject files with an unsupported 
major version.

---

## 9. Example File

See [`examples/sample_flight.jsonl`](examples/sample_flight.jsonl) for a 
complete annotated example.

---

## 10. Reference Implementation

The reference implementation is part of the 
[Scales platform](https://flyscales.com). 
The server writes STF natively; readers are available in Python.
