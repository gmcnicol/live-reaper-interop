# Host validation handoff

**Status: blocked; no DAW tests run.** This procedure supports point-in-time, best-effort imports into fresh destinations only. Persistent clip identity, history and existing-project updates are not required.

## Observed blocker

On 2026-11-17 UTC, this worker reported `Linux 7.0.0-43-generic x86_64`. No `reaper`, `REAPER`, `ableton`, `Live`, `wine` or `wine64` executable was found on PATH. A search to depth four under `/opt`, `/usr/local` and `/home/gareth/.local` found no matching installation. These checks do not describe other machines or prove no installation exists elsewhere.

Needed to run: an operator with disposable projects in named REAPER and Live builds/editions, permission to run Lua ReaScript and a diagnostic Python Remote Script, and access to the fixture files. Record Live's embedded Python version, not an external interpreter. No hosts, scripts or codecs were installed by this task.

## Evidence and detailed procedures

The research branches remain separate and unmerged. Their pinned reports contain cited API evidence and detailed tests:

- [Live capabilities](https://github.com/gmcnicol/live-reaper-interop/blob/1b6c906e4034ba0c75646f7705d3fd688f9ab812/docs/research/live-capabilities.md#minimal-real-host-validation-procedure): use environment, creation, adjacency, persistence and recovery sections; omit the identity section.
- [REAPER capabilities](https://github.com/gmcnicol/live-reaper-interop/blob/2d8dd9297756174ce1368249c3e919032328bb67/docs/research/reaper-capabilities.md#minimal-real-host-validation-procedure--not-run): use environment, geometry, sound and recovery tests; omit GUID/tag lifecycle tests.
- [MIDI/CC and audio-format procedures](https://github.com/gmcnicol/live-reaper-interop/blob/54cf2aad828d929d6fb6aa2188755a81eeb90422/docs/research/midi-audio-format-capabilities.md#reproducible-real-host-validation-procedures-not-executed): use the fixed MIDI event table and shared-format calibration. This report supplies the audio coordinates below.

Documentation, UI operations and inspected Python source are not successful Python host tests. Log missing APIs as blocked; do not invent setters or substitute Max for Live results for Python results.

## Small reproducible audio fixture

Run this in an empty disposable directory with Python 3. It creates a 10-second stereo, 16-bit PCM WAV at 48 kHz. It refuses to overwrite an existing file and verifies the written frames. Generating this fixture is not a host integration test.

```sh
python3 - <<'PY'
import hashlib
import struct
import wave
from pathlib import Path

path = Path("transients.wav")
rate, frames = 48000, 480000
pulses = (0, 1, 529, 1153, 48001, 96007, 144013, 479998)
samples = bytearray(frames * 4)
for frame in pulses:
    struct.pack_into("<hh", samples, frame * 4, 24576, -8192)
with path.open("xb") as output:
    with wave.open(output, "wb") as fixture:
        fixture.setparams((2, 2, rate, frames, "NONE", "not compressed"))
        fixture.writeframes(samples)
with wave.open(str(path), "rb") as fixture:
    assert (fixture.getnchannels(), fixture.getsampwidth()) == (2, 2)
    assert (fixture.getframerate(), fixture.getnframes()) == (rate, frames)
    assert fixture.readframes(frames) == samples
print(path, hashlib.sha256(path.read_bytes()).hexdigest())
PY
```

Local checks passed for generated frame metadata/payload, refusal to overwrite existing media, and audio/MIDI adjacency arithmetic. Generated WAV SHA-256: `65f6f2ffda106cf2d2593f992c1e6ea0fb61a77e32bf0d09e968e86064b8d75c`. These checks did not involve either DAW.

## Run the smallest host experiment

1. Save an empty disposable baseline in each host. Use 120 BPM, 4/4, zero timeline origin, 48 kHz and normal playback speed. Disable effects, fades, looping, warp and other processing. Keep originals untouched.
2. Import the fixture through each proposed adapter API. Set timeline start to `123457/48000` seconds, source offset to `96007/48000` seconds, and visible duration to `50003/48000` seconds. Convert timeline seconds to quarter-note beats with `beats = seconds * 2` where required. Keep source seconds separate.
3. Read every boundary and the source reference back. REAPER candidates are item `D_POSITION`, `D_LENGTH` and `B_LOOPSRC`, plus take `D_STARTOFFS` and `D_PLAYRATE`. In Live, probe `create_audio_clip(path, position)` only if available; read `start_time`, `end_time`, source-region markers, `file_path` and warp state. Wait for deferred warp changes before interpreting source bounds. Read-only timeline edges are not setters.
4. Verify the audible segment against the known pulses, not only API values. Record requested and observed start, end and source offset. For PCM, report fractional-frame error as `actual_seconds * 48000 - expected_frame`; one frame is a target to measure, not an achieved guarantee.
5. Place an adjacent clip at `(123457 + 50003)/48000` seconds and a gapped repeat on another named track. Try both insertion orders; move/recreate a clip independently. After each operation read all neighboring clips to detect truncation or overwrite.
6. Save a new copy, close the host, reopen, enumerate again and repeat boundary, source, count and audible checks. Verify that original audio bytes are unchanged.
7. On a separate disposable copy, try a missing source and a failure after one successful creation. Record exceptions, residual clips and undo/redo results. If rollback fails, close without saving and reopen the untouched baseline. Do not claim atomic recovery from an undo-group call alone.

## MIDI/CC and other audio formats

Use the exact MIDI fixture table in the pinned MIDI/format report: channels 1/2/16, polyphonic notes, non-grid note/CC times, sustain before/inside clip bounds, visible source region `[1.125, 4.5]` beats, arrangement start `9.1375` beats and an adjacent clip at `12.5125` beats. Record source-boundary and event-chase behavior instead of assuming it.

For each host, prove both API write and full read-back of note pitch/time/duration/velocity/channel and timed CC channel/controller/value. Preserve sustain as CC data. A missing channel field, remapped channels or unavailable CC access is a failed or blocked required capability, not a notes-only pass. Live's CC/channel path remains unestablished by research. Save/reopen and repeat. Report unsupported or uninspectable event types honestly.

For non-WAV files, use existing known test encoders/assets; do not install a new tool or convert production sources for the bridge. Test each claimed host/OS/format combination, starting with documented candidate families AIFF, FLAC, Ogg Vorbis and MP3. Verify original reference versus decoded cache, source zero, seek accuracy and save/reopen. For lossy files compare seeks with each host's full decoded reference, not bitwise equality with pre-encoding PCM. No filename-extension whitelist establishes fidelity.

Run both REAPER → Live → fresh REAPER and Live → REAPER → fresh Live only when the necessary primitives are available. Otherwise mark the missing step blocked. A successful WAV-only or notes-only experiment does not verify the expanded milestone. Best-effort omissions do not prove omitted features work.

## Current runtime matrix

| Test | REAPER | Live |
| --- | --- | --- |
| Non-grid audio start, duration and non-zero source offset | Unverified | Unverified |
| Audible source segment and measured precision | Unverified | Unverified |
| Adjacent clips and move/recreate safety | Unverified | Unverified |
| Save/reopen and original-media integrity | Unverified | Unverified |
| MIDI notes, original channels and timed CC | Unverified | Unverified |
| Shared formats, cache paths and seek behavior | Unverified | Unverified |
| Missing media, partial failure and undo/recovery | Unverified | Unverified |
| Both fresh-destination round trips | Unverified | Unverified |

## Evidence record to return

Copy this record per subtest; attach logs and disposable project copies rather than replacing expected values with observations.

```text
Test / direction / date / operator:
OS / host exact version and edition / embedded interpreter / decoder:
Diagnostic script revision / exact API calls and units:
Fixture path / SHA-256 / sample rate / frame count or MIDI event reference:
Expected start / end / source offset or note/CC events:
Observed before / after / after reopening:
Measured errors / audible or raw MIDI evidence / neighbor changes:
Omissions / uninspectable fields / exceptions / recovery outcome:
Result: PASS | FAIL | BLOCKED | UNRUN
Evidence links:
```

This handoff completes the task's explicit-blocker path, not host validation. Adapter and contract decisions may proceed conditionally; no tested version, precision or MIDI/CC guarantee is established here.
