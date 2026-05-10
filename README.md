# Scales Trajectory Format (STF)
The Scales Trajectory Format (STF) — open standard for body-flight performance data


## Why STF?

Indoor skydiving has no common language for performance data. Every 
tunnel, every coach, every athlete tracks sessions differently — or 
not at all. STF is our proposal to change that.

We believe that open, structured flight data is the foundation for 
everything that comes next: objective coaching, athlete progression 
tracking, cross-tunnel comparison, and eventually, federation-level 
competition analytics.

STF is designed to be simple enough that anyone can implement it, 
and open enough that no single platform controls it.

## What is STF?

A lightweight, time-series data format for recording position, 
orientation, and quality metrics of a body-flight session inside a 
wind tunnel. Plain JSONL — no binary encoding, no special tools needed.

## Format Overview

Each STF file is a JSONL (newline-delimited JSON) file:

- **Line 1:** Header object (metadata)
- **Lines 2+:** Frame objects (sensor data at time `t`)

### Header
```json
{
  "schema_version": "1.0",
  "flight_id": "2026-04-19_playbox_il_f01",
  "tunnel_id": "playbox_il",
  "pilot_id": "...",
  "coordinate_frame": "tunnel",
  "units": "meters"
}
```

### Frame
```json
{
  "t": 1234567890.123,
  "devices": {
    "tag_hip": {
      "position": {"x": 1.2, "y": 0.5, "z": 2.1},
      "orientation": {"qw": 1.0, "qx": 0.0, "qy": 0.0, "qz": 0.0},
      "quality": 0.98
    }
  }
}
```

## Design Goals

- **Open:** The spec is public. Anyone can implement a reader or writer.
- **Simple:** Plain JSONL — readable by humans, parseable by any language.
- **Extensible:** New device types and fields can be added without 
  breaking existing parsers.
- **Neutral:** STF is not tied to any hardware vendor or platform.

## Status

STF v1.0 is under active development as part of the 
[Scales](https://flyscales.com) platform — a performance tracking 
system for indoor skydiving. We are proposing STF as a community 
standard and welcome feedback from tunnel operators, coaches, 
athletes, and federations.

## Get Involved

Have thoughts on the format? Open an issue or start a discussion.
We're especially interested in feedback from:
- Tunnel operators
- IBA / FAI technical committees  
- Coaches and athletes working with performance data

## License

The STF specification is released under 
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
The reference implementation is part of the Scales platform.
