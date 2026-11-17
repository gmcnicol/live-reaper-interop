# Project scope and feasibility plan

## Objective

Transfer editable audio and MIDI arrangement structure between REAPER and Ableton Live, in either direction. Preserve where clips sit, which portions of audio sources they use, and MIDI notes plus CC events.

This document describes intended behaviour, not implemented functionality or a guarantee of API support. The [resolved first-transfer scope](https://github.com/gmcnicol/live-reaper-interop/issues/9) expands the original WAV-only proposal to shared audio formats and MIDI notes plus CC; those capabilities require additional research and host validation.

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

## Milestone 1: audio and MIDI interchange into a fresh destination

Supported input targets, subject to capability evidence:

- Original audio sources in any format readable by both target host versions, not only WAV. No automatic conversion or rendering.
- One constant project tempo, one 4/4 time signature and a shared timeline origin.
- Audio at normal playback speed, with no warping, looping or reversal.
- One active source take per REAPER item.
- Flat, single-kind audio or MIDI tracks, with no overlapping arrangement clips on the same track. Polyphonic MIDI notes inside a clip are allowed.
- Non-negative timeline starts and a shared zero.
- No fades or crossfades.
- Editable MIDI notes with pitch, onset, duration, velocity and channel, plus timed CC events with channel, controller number and value. Sustain remains CC data rather than rewritten note lengths.

Transfer track names and order, clip names, audio media references, timeline starts, visible durations and audio source offsets. Preserve non-grid positions, adjacent clips, gaps and repeated audio-source references.

Use faithful best-effort transfer: import supported material and report omitted items or MIDI event types. Skip a clip when its timing or source content cannot be represented correctly; do not silently flatten, render, convert, retime or guess. Preflight before mutation. Invalid transfer data, unresolved required media and unsafe destination changes remain blocking errors. Partial-import recovery details belong to the later contract decision.

MIDI pitch bend, program changes, aftertouch, release velocity, SysEx and MPE/per-note expression are not first-milestone targets. Instruments and their sound are not transferred.

Import into a new project/set or an explicitly empty target arrangement on every leg. A return import creates a separate arrangement rather than updating the original. Preserve the exported manifest as the record of the transfer.

## Proposed interchange data

The schema is to be finalised after the feasibility experiment.

| Data | Intended meaning |
| --- | --- |
| Format version | Contract version; reject unknown incompatible versions |
| Origin application and version | Reproduce compatibility findings |
| Constant tempo and time signature | Define the supported musical timeline |
| Timeline origin | Align zero between hosts |
| Track reference, name and order | Reconstruct track layout within this snapshot |
| Item track reference | Associate each clip with its track within this snapshot |
| Source identity, path, sample rate and frame count | Resolve and validate source audio |
| Timeline start/end in seconds and quarter-note beats | Make timing intent and conversion explicit |
| Source offset in seconds/sample frames | Identify the first audible source position |
| Clip name | Preserve human-readable labels |
| MIDI notes and CC events | Preserve the agreed editable musical data, including channels; event coordinates and clip-boundary semantics remain to be decided |

Use explicit units and full numeric precision. Treat elapsed seconds as authoritative for the first audio milestone; beat positions are derived from the constant tempo and provide a consistency check. Do not use displayed bars/beats strings as interchange coordinates.

At constant tempo, with a shared zero:

- quarter-note beats = seconds × BPM / 60
- seconds = quarter-note beats × 60 / BPM

Convert the start and end boundaries independently, then derive duration. Source offset is a separate coordinate from timeline start. A clip at 30 seconds can begin playing 90 seconds into its source recording.

For REAPER, use its time-map functions when exporting musical coordinates. Variable tempo and tempo ramps need an explicit later contract and must be rejected by the first milestone.

Paths are local references, not portable identities. Prefer relative paths when media is explicitly collected beside a manifest; allow path remapping when moving between machines. Confirm source identity before substituting a missing file. Do not modify source recordings.

## Point-in-time boundary

Each import reconstructs the current exported snapshot in a fresh destination, on a best-effort basis. It does not reconcile with an earlier import or modify the original project.

Existing-project updates, transfer history, persistent clip matching, identity sidecars, idempotent reimport into existing arrangements, deletion propagation and conflict/merge workflows are outside this effort. No plan for those features is required.

## Acceptance checks

Use disposable projects with original media kept intact. Test both REAPER → Live → REAPER and Live → REAPER → Live.

The audio fixture should include multiple tracks, gaps, adjacent clips, repeated references to one recording, non-grid positions and non-zero source offsets. Include recognisable transients to reveal source-offset errors. Begin calibration with PCM WAV, then validate the claimed shared audio-format matrix; WAV-only success does not satisfy the expanded milestone.

Add MIDI fixtures with polyphonic notes, channel distinctions, non-grid note and CC timing, clip cuts and sustain CC events. Validate editable read-back after save/reopen. MIDI API access, timing units and boundary handling remain unverified.

Verify:

- Track order, names and supported clip counts survive.
- No imported item unexpectedly overlaps or truncates another.
- Start, end and source offsets match the PCM fixture sample grid within one sample at the fixture's sample rate, including after reopening. Investigate decoder/seek limits separately for other claimed audio formats.
- The audible source segment is correct; this milestone introduces no resampling or processing.
- Missing required media blocks mutation. Unsupported material produces a faithful partial transfer with a clear omission report, not silent approximation.
- The original files and source projects are intact.

One-sample accuracy is an acceptance target to measure, not an established guarantee. Record the achieved tolerance and actual API limitations.

## Later work

- MIDI event families beyond notes and CC, including pitch bend, program changes, aftertouch, release velocity, SysEx and per-note expression.
- Tempo and time-signature maps, including tempo ramps.
- Looping and repeated source regions.
- Warping/stretch markers and playback-rate mapping.
- Overlapping REAPER items, lanes and comping.
- Fades and crossfades.
- Optional media collection and packaging.

Stretch algorithms can sound different even when timing maps translate. Any future render/bake option must be explicit.

## Outside the initial scope

Plug-in state, instruments, device chains, routing, sends, mixer levels, pan, host automation, Session View scenes and live synchronisation. MIDI CC events are in scope and are distinct from host automation.

## API references

- [REAPER ReaScript API](https://www.reaper.fm/sdk/reascript/reascripthelp.html): item position/length, take source offsets and time-map conversion.
- [Live Track object](https://docs.cycling74.com/apiref/lom/track/): arrangement clip enumeration, creation and duplication.
- [Live Clip object](https://docs.cycling74.com/apiref/lom/clip/): arrangement boundaries, source bounds and warp state.
- [Ableton third-party Remote Scripts](https://help.ableton.com/hc/en-us/articles/209072009-Installing-third-party-remote-scripts): Python 3 and installation guidance.

These references guide the experiment. A supported-version matrix should be published only after testing actual host installations.
