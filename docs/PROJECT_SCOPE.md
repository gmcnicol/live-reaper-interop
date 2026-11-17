# Project scope and feasibility plan

## Objective

Transfer editable arrangement structure between REAPER and Ableton Live, in either direction. Preserve where audio items sit and which portions of their source recordings they use.

This document describes intended behaviour, not implemented functionality or a guarantee of API support.

## Architecture

Two host adapters share a versioned JSON arrangement manifest:

- REAPER: Lua ReaScript for export and import.
- Ableton Live: Python Remote Script for export and import of Arrangement View clips.

The manifest describes arrangement data rather than either DAW's internal project format. A shared folder is sufficient for the initial workflow. Direct editing of RPP or ALS project files is not the initial approach.

The Python implementation must be tested against a named Live version. Live's documented object model is useful capability evidence, but does not establish that every operation is available or behaves identically through a particular Python Remote Script environment.

## Milestone 0: prove the Live operations

Before fixing the interchange schema or building a full importer, run a small experiment in the installed Live version:

1. Enumerate Arrangement View clips and read their file paths and arrangement boundaries.
2. Create an audio clip from an existing file at a chosen arrangement position.
3. Disable warping and set a non-zero source offset and a visible duration shorter than the source.
4. Verify that changing source bounds does not unexpectedly shift the arrangement start.
5. Read back arrangement boundaries and source bounds, including after saving and reopening.
6. Recreate or duplicate a clip at another arrangement position when direct movement is unavailable.
7. Exercise the same operations on adjacent clips to detect unintended overwrite or truncation.
8. Verify that a failed import can be undone or safely abandoned without damaging existing material.

Use a source recording containing recognisable events at known sample positions. Comparing only API values is insufficient: confirm that the expected source segment actually plays.

Record exact Live, Python and REAPER versions, API calls used, observed units, read-back results and any limitations. The documented read-only arrangement start/end properties make exact placement and trimming a specific feasibility gate.

If the necessary operations cannot be performed through the selected API, document the blocker and revisit the adapter approach before promising a complete bridge.

## Milestone 1: audio interchange into a fresh destination

Supported input:

- Local PCM WAV sources in formats supported by both hosts.
- One constant project tempo, one 4/4 time signature and a shared timeline origin.
- Audio at normal playback speed, with no warping, looping or reversal.
- One active source take per REAPER item.
- Flat audio tracks, with no overlapping items on the same track.
- Non-negative timeline starts.

Transfer track names and order, clip names, media references, timeline starts, visible durations and source offsets.

Preflight unsupported constructs before modifying the destination. Do not silently discard them, overwrite other clips, render them or change their timing. Report the affected tracks/items and the reason. Users can prepare a supported project copy explicitly.

Import into a new project/set or an explicitly empty target arrangement. Preserve the exported manifest as the record of the transfer.

## Proposed interchange data

The schema is to be finalised after the feasibility experiment.

| Data | Intended meaning |
| --- | --- |
| Format version | Contract version; reject unknown incompatible versions |
| Transfer/project identity | Identify related exports without relying on filenames |
| Origin application and version | Reproduce compatibility findings |
| Constant tempo and time signature | Define the supported musical timeline |
| Timeline origin | Align zero between hosts |
| Track identity, name and order | Reconstruct track layout |
| Item identity and track identity | Describe item membership and future matching |
| Source identity, path, sample rate and frame count | Resolve and validate source audio |
| Timeline start/end in seconds and quarter-note beats | Make timing intent and conversion explicit |
| Source offset in seconds/sample frames | Identify the first audible source position |
| Clip name | Preserve human-readable labels |

Use explicit units and full numeric precision. Treat elapsed seconds as authoritative for the first audio milestone; beat positions are derived from the constant tempo and provide a consistency check. Do not use displayed bars/beats strings as interchange coordinates.

At constant tempo, with a shared zero:

- quarter-note beats = seconds × BPM / 60
- seconds = quarter-note beats × 60 / BPM

Convert the start and end boundaries independently, then derive duration. Source offset is a separate coordinate from timeline start. A clip at 30 seconds can begin playing 90 seconds into its source recording.

For REAPER, use its time-map functions when exporting musical coordinates. Variable tempo and tempo ramps need an explicit later contract and must be rejected by the first milestone.

Paths are local references, not portable identities. Prefer relative paths when media is explicitly collected beside a manifest; allow path remapping when moving between machines. Confirm source identity before substituting a missing file. Do not modify source recordings.

## Milestone 2: return changes to an existing project

The goal is to export from one host, rearrange in the other and apply the return transfer to the original project without duplicating unchanged items.

First prove a persistent bridge identity strategy. Do not rely on track names, clip names, array indices or runtime Live object IDs as durable identity on their own. A sidecar mapping may be needed, but its association with clips must survive the supported edits.

Define and test these cases explicitly:

- Save, close and reopen either host.
- Move a clip in time or to another track.
- Rename or reorder tracks and clips.
- Trim a clip or slip its source content.
- Split a clip into independently identified descendants.
- Duplicate a clip with a new identity.
- Add or delete a clip.
- Reimport the same return manifest twice.
- Detect unrelated edits made in both hosts since their common export.

Ambiguous identity or conflicting edits must produce a reviewable report rather than guessed matches. Deletion propagation and replacing existing material require explicit import choices. The first interchange milestone does not depend on solving automatic merge.

## Acceptance checks

Use disposable projects with original media kept intact. Test both REAPER → Live → REAPER and Live → REAPER → Live.

The fixture should include multiple tracks, gaps, adjacent clips, repeated references to one recording, non-grid positions and non-zero source offsets. Include recognisable transients to reveal source-offset errors.

Verify:

- Track order, names and supported clip counts survive.
- No imported item unexpectedly overlaps or truncates another.
- Start, end and source offsets match the chosen fixture sample grid within one sample at the fixture's sample rate, including after reopening.
- The audible source segment is correct; this milestone introduces no resampling or processing.
- Missing media and unsupported constructs are reported before mutation.
- The original files and source projects are intact.

One-sample accuracy is an acceptance target to measure, not an established guarantee. Record the achieved tolerance and actual API limitations.

For milestone 2, also verify idempotent reimport, persistent identity and conflict reporting.

## Later work

- MIDI clips and ordinary note events, with explicit treatment of unsupported per-note expression.
- Tempo and time-signature maps, including tempo ramps.
- Looping and repeated source regions.
- Warping/stretch markers and playback-rate mapping.
- Overlapping REAPER items, lanes and comping.
- Fades and crossfades.
- Optional media collection and packaging.

Stretch algorithms can sound different even when timing maps translate. Any future render/bake option must be explicit.

## Outside the initial scope

Plug-in state, instruments, device chains, routing, sends, mixer levels, pan, automation, Session View scenes and live synchronisation.

## API references

- [REAPER ReaScript API](https://www.reaper.fm/sdk/reascript/reascripthelp.html): item position/length, take source offsets and time-map conversion.
- [Live Track object](https://docs.cycling74.com/apiref/lom/track/): arrangement clip enumeration, creation and duplication.
- [Live Clip object](https://docs.cycling74.com/apiref/lom/clip/): arrangement boundaries, source bounds and warp state.
- [Ableton third-party Remote Scripts](https://help.ableton.com/hc/en-us/articles/209072009-Installing-third-party-remote-scripts): Python 3 and installation guidance.

These references guide the experiment. A supported-version matrix should be published only after testing actual host installations.
