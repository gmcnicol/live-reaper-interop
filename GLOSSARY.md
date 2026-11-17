# Arrangement Interchange

Language for transferring editable arrangement structure between REAPER and Ableton Live.

## Language

**Fresh-destination transfer**:
A transfer that reconstructs an arrangement in a new or explicitly empty destination rather than changing an existing arrangement.
_Avoid_: Merge, update

**Fresh-destination round trip**:
A transfer from one DAW to the other and back, with a fresh destination on each import. The return is a separate arrangement, not an update to the original.
_Avoid_: Synchronisation, return update

**Best-effort transfer**:
A transfer that preserves faithfully representable material and reports unsupported material it omits. It does not silently approximate timing, source content or musical events.
_Avoid_: Lossless transfer, approximate transfer
