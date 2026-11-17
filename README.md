# Live–REAPER Interop

An arrangement bridge for moving editable audio items between REAPER and Ableton Live's Arrangement View, in both directions.

The intended workflow is to export an arrangement, edit its clip placement in the other DAW, and transfer the arrangement back while preserving source references, cuts and timing.

**Status: planning and feasibility. No working importer or exporter is included yet.**

## First milestone

Prove a round trip using unwarped audio at one constant tempo:

- Preserve track names and order.
- Preserve item start positions, visible durations and offsets into source recordings.
- Reference the original audio without rendering or consolidating it.
- Import into a fresh destination arrangement in either DAW.
- Report unsupported material and missing media before changing the destination.

Updating an existing project without duplicates is a subsequent milestone. Persistent item identity, including after saves, splits and duplication in Live, needs to be proven first.

## Proposed design

A Lua ReaScript adapter and a Python Remote Script adapter exchange a versioned JSON manifest and references to media files. Each adapter translates between the shared arrangement model and its host's API.

No network service is required for the first version. Media stays local; transfers between computers will need explicit media collection or path remapping.

## Scope

Plug-ins, instruments, routing, mixer settings and automation are outside the initial scope. MIDI transfer, tempo changes, looping, stretching and fades are later work.

See [Project scope and feasibility plan](docs/PROJECT_SCOPE.md) for the proposed contract, limitations and acceptance checks.

## Licence

[MIT](LICENSE).
