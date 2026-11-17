# Live–REAPER Interop

An arrangement bridge for moving editable audio and MIDI clips between REAPER and Ableton Live's Arrangement View, in both directions.

The intended workflow is to export an arrangement, edit its clip placement in the other DAW, and transfer the arrangement back while preserving source references, cuts and timing.

**Status: planning and feasibility. No working importer or exporter is included yet.**

## First milestone

Prove a fresh-destination round trip at one constant tempo, using unwarped audio and editable MIDI notes plus CC messages:

- Preserve track names and order.
- Preserve item start positions, visible durations and offsets into source recordings.
- Reference original audio in formats readable by both target hosts, without conversion, rendering or consolidation.
- Preserve MIDI note timing, pitch, duration, velocity and channels, plus timed CC messages including sustain.
- Import into a fresh destination arrangement in either DAW.
- Import faithfully representable material and report omissions; do not silently approximate unsupported content.
- Stop on invalid transfer data, unresolved required media or unsafe destination changes.

Each import is a point-in-time, best-effort snapshot into a fresh destination. Existing-project updates, transfer history, persistent clip matching and merge behaviour are outside this effort.

## Proposed design

The proposed Lua ReaScript and Python Remote Script adapters exchange a versioned JSON manifest and references to media files. Each adapter translates between the shared arrangement model and its host's API. MIDI/CC access and shared audio-format support need investigation alongside the audio placement feasibility gate.

No network service is required for the first version. Media stays local; transfers between computers will need explicit media collection or path remapping.

## Scope

Plug-ins, instruments, routing, mixer settings and host automation are outside the initial scope; MIDI CC messages are in scope. Tempo changes, looping, stretching, fades and MIDI event families beyond notes and CC are later work.

The [resolved first-transfer scope](https://github.com/gmcnicol/live-reaper-interop/issues/9) is the product target, not verified host support. See [GLOSSARY.md](GLOSSARY.md) for fresh-destination and best-effort terminology.

See [Project scope and feasibility plan](docs/PROJECT_SCOPE.md) for the proposed contract, limitations and acceptance checks.

## Licence

[MIT](LICENSE).
