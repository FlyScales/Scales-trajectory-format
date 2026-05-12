# Changelog

## v1.0.0 — May 2026
- Initial specification release
- Header schema: schema_version, flight_id, frame_format, tunnel_id,
  pilot_id, coordinate_frame, units, created_by
- Frame schema: t, devices (position, orientation, quality, raw_distances)
- Quality flags: good, interpolated, dead_reckoning, dropped
- Coordinate frame: tunnel-relative, flyer-facing-operator perspective
- Derived fields policy: velocity, speed, euler angles computed on read
