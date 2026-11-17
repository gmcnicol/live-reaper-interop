# Live Arrangement API and identity capabilities

Research for [issue #2: Establish Live Arrangement API and identity capabilities](https://github.com/gmcnicol/live-reaper-interop/issues/2), within [the Wayfinder map](https://github.com/gmcnicol/live-reaper-interop/issues/1).

**Retrieved: 2026-11-17 UTC** (worker clock). **Status: evidence investigation complete; host feasibility not proven.** No Ableton Live or REAPER operation was run. No adapter, schema, or supported-host matrix is delivered.

## Answer

The proposed Python Remote Script route is credible enough for a disposable host experiment, not yet a validated importer. The official Live Object Model (LOM) documents Arrangement enumeration, audio file references, readable arrangement edges, writable source-region/warp controls, file-backed creation, duplication to a specified beat, and deletion. It does **not** document writable arrangement start/end properties or a durable clip UUID. [Track][track], [Clip][clip], [Live API overview][overview]

There is separate, concrete **Python source evidence**: an independently authored Remote Script reads `arrangement_clips`, `start_time`, and `end_time`, and calls `create_audio_clip`, `duplicate_clip_to_arrangement`, and `delete_clip`. This demonstrates implemented Python call sites, **not successful execution here**, nor correctness on a named installed host. The controller-source mirror also contains Arrangement duplication and undo-grouping calls, but it is unofficial and decompiled. [Python Track][py-track], [Python Clip][py-clip], [mirror provenance][mirror-readme], [mirrored duplication][mirror-actions], [mirrored undo][mirror-undo]

The hard gates are independent placement versus source trimming, identification of the first audible source sample after Arrangement edits, adjacency without collateral changes, deferred warp updates, save/reopen, and error recovery. Existing-project return updates have an additional identity gate: documented Max IDs are explicitly not persisted. That is evidence **against using those IDs**, not proof that every possible Python identity mechanism is absent. [Clip][clip], [ID lifetime][overview], [LiveAPI.id][js-api]

## Evidence boundaries and method

- The ticket and current map define the question. `README.md` and `docs/PROJECT_SCOPE.md` were read as hypotheses, not capability evidence. Only this findings file changes in Git.
- Primary evidence means official Ableton/Cycling ’74 documentation or a project's own source for claims about that project's implementation. An adapter's source is primary evidence of its calls, not an Ableton API support promise.
- Context7 discovery/query was used, then official pages were fetched directly. Its Python hits included unofficial sources; their reputation labels were not treated as vendor endorsement.
- Ableton's Remote Script HTML article returned HTTP 403. Its first-party [Help Center JSON representation][remote-json] was accessible and supplied the article body. The recovery article was also read through its [first-party JSON representation][recovery-json]. Other cited official pages were accessible directly.
- Environment limit supplied to this worker: no `reaper` or `wine` on PATH, and no installation found in shallow `/opt`, `/usr/local`, or `/home/gareth/.local` checks. This is not evidence about the human's other machines. No software was installed; no host testing was attempted or inferred from source, mocks, or documentation.

Evidence labels used below:

| Label | Meaning |
| --- | --- |
| **D** | Officially documented LOM/Max or application behavior; not necessarily Python exposure. |
| **P** | Specific Python call site inspected in pinned source; no execution claim. |
| **I** | Inference or candidate experiment, not a documented guarantee. |
| **U** | Unestablished by inspected evidence. |
| **R** | Runtime-tested on a named host. **There are no R results in this report.** |

Confidence describes the stated evidence, not the probability of a successful bridge. **High**: explicit official statement. **Medium**: static Python call sites or constrained inference. **Unknown**: behavior needs a host test. Absence from the inspected reference is not proof that a private API cannot exist.

## Documented versions, editions, and installation

| Evidence | Exact documented boundary | Consequence |
| --- | --- | --- |
| Current Cycling ’74 LOM index | “This document refers to Ableton Live version **12.4.5**.” [LOM index][lom] | The current Track/Clip reference is not a promise for every Live 11/12 build. |
| `Track.arrangement_clips` | “Available since Live **11.0**”; get/observe list. [Track][track] | Establishes the documented enumeration introduction, not the complete import feature floor. |
| File-backed audio creation announcement | Live **11.3.25**, **May 8, 2024**, Max for Live API. The release notes literally name `Track.create_audio_track` and `ClipSlot.create_audio_track` while describing clip creation. Current LOM instead spells both clip-creation methods `create_audio_clip`. [11 release notes][rel11], [Track][track], [ClipSlot][slot] | Record the documentation naming discrepancy; do not call the release-note spelling as a clip API. This announces the feature in the 11 branch, not a measured Python minimum or a proven earliest 12.x release. |
| Older duplication defects | Live **11.3.2** notes fixes for a crash duplicating an Arrangement clip at its original start and for accepting negative destination times. [11 release notes][rel11] | “Live 11+” is too broad a safety guarantee. |
| Later Arrangement API defects | Live **12.1** notes a `live.path` resolution fix for `Track.arrangement_clips`; **12.3** notes a `duplicate_clip_to_arrangement` crash fix; **12.4** notes both same-track Arrangement duplication and Arrangement creation crash fixes. [12 release notes][rel12] | Enumerate and test exact builds. These are vendor-reported fixes, not observed failures or proof that every earlier build fails. |
| Python generation | Live **10 and earlier: Python 2**; **11 and later: Python 3**. [Remote Script installation][remote] | A custom script must target the host's embedded Python, not the Linux worker's Python. |
| Embedded Python patch version | Live **12.1** updated its interpreter to **3.11.6**; compiled `.pyc` scripts must be recompiled accordingly. [12 release notes][rel12] | This exact patch version is evidence for 12.1, not all later Live builds. Record the actual host interpreter. |
| User Library installation | Since **10.1.13**, scripts can live in `User Library/Remote Scripts`; launch Live, select the script in Preferences → MIDI → Control Surface, and assign relevant device ports. [Remote Script installation][remote] | Default User Library: Windows `\Users\[username]\Documents\Ableton\User Library`; macOS `~/Music/Ableton/User Library`. The article covers controller installation, not whether a file-only script can omit MIDI ports. |
| Support and editions for Python route | Ableton explicitly declines technical support for scripts not included with Live; the installation article gives no per-edition Arrangement/Python feature guarantee and does not state a Max for Live requirement. [Remote Script installation][remote] | Do not infer Suite-only licensing from LOM docs, or claim tested Intro/Lite/Standard support from silence. Record edition in every experiment. |
| Max for Live alternative | Live **12 manual**: bundled with **Suite**, or **Standard with the Max for Live add-on**; no separate installation required. Choosing an external Max editor requires setting its path and restarting Live. [Max for Live][max-live] | This is an explicit edition requirement for the Max route, separate from Remote Scripts. |
| Python source provenance | Ableton.js pinned at `63bf873c5f1063a67011c6665d892833fcd54701`; README says **Live 11 or later**, not a per-operation build matrix. The other repository labels itself unofficial **Live 12.4** sources. [adapter README][py-readme], [mirror README][mirror-readme] | Neither source establishes our exact supported host. Decompiled file headers can reflect older bytecode; do not derive the host interpreter from them. |

No complete minimum version/edition for the proposed Python importer can be established from these sources alone. The feature announcement, LOM's current version, bug-fix versions, and adapter README prerequisite are different evidence, not interchangeable support guarantees.

## Capability/evidence matrix

All rows have **runtime status U / not tested**. “Python evidence” deliberately separates direct call sites from generic forwarding or LOM-only evidence.

| Operation | Official documented capability or limit | Python evidence | Confidence and remaining gate |
| --- | --- | --- | --- |
| Enumerate tracks and Arrangement audio clips | `Song.tracks`; `Track.arrangement_clips` is a get/observe list of Arrangement clip IDs, since 11.0. `Clip.is_audio_clip` distinguishes audio; take-lane clips also count as Arrangement clips, with `is_take_lane_clip` available in current LOM. [Song][song], [Track][track], [Clip][clip] | **P:** `Song.get_tracks`, `Track.get_arrangement_clips`, and clip serialization access these host objects. [Python Song][py-song], [Python Track][py-track], [Python Clip][py-clip] | **High D / medium P.** Test count/order, repeated file references, hidden/group tracks, and exclusion of take-lane material; do not assume ordering is a stable identity. |
| Read media reference | `Clip.file_path` is get-only and audio-only. `sample_length` gives samples; `sample_rate` is readable. [Clip][clip] | **P, generic only:** adapter `get_prop` falls back to Python `getattr`; its clip serializer does not explicitly read the file path or sample metadata. [Python interface][py-interface], [Python Clip][py-clip] | **High D / unknown specific P.** Test readable/missing/unicode paths, collected media, and exact sample metadata. A pathname is not a durable source or clip identity. |
| Read timeline start/end | `start_time` is get/observe, explicitly global-song beats and an Arrangement offset. `end_time` is get/observe and the rightmost Arrangement edge. Its entry does not independently restate units. [Clip][clip] | **P:** serializer directly reads both properties. [Python Clip][py-clip] | **High D / medium P.** Treat end-time beats as a hypothesis to verify against tempo and UI; no writable-edge contract. Do not use Session `start_time` semantics as Arrangement semantics. |
| Read/write source region and offset | `loop_start`/`loop_end` are get/set/observe: for unlooped clips, clip start/end; unwarped audio uses **seconds**, with zero at the file's first sample. MIDI/warped audio uses clip beats. `start_marker`/`end_marker` are separately writable and described as beats, independent of loop state. [Clip][clip] | **P, generic only:** forwarding uses `getattr`/`setattr`; no inspected explicit audio offset/trim implementation or test. [Python interface][py-interface] | **High D / unknown actual Arrangement mapping.** Do not equate a marker or `loop_start` with the first audible source position after every Arrangement left-edge trim. Prove that mapping and setter order on the host. |
| Determine/change warp state | `warping` and `warp_mode` are writable, audio-only. **Setting `warping` is internally deferred**, so API calls from one event may execute in an unexpected effective order. Unwarped audio cannot loop. `length` “makes no sense for unwarped audio clips.” [Clip][clip] | **P:** explicit warp-mode access; `warping` would use generic forwarding. [Python Clip][py-clip], [Python interface][py-interface] | **High D / medium mode P / unknown sequencing.** Wait for observed/read-back warp-off before interpreting bounds as seconds. Never use `Clip.length` as the unwarped visible duration. |
| Create file-backed Arrangement audio | `Track.create_audio_clip(file_path, position)` references an absolute valid supported audio file. Position range `[0., 1576800]`; reports errors for non-audio, frozen, or recording tracks. There is **no duration or source-offset parameter**. The entry does not explicitly label position units. [Track][track] | **P:** wrapper directly calls it and serializes the returned clip. [Python Track][py-track] | **High D / medium P.** Verify position units (beats are a candidate), return value, failure channel, initial extent, and auto-warp preferences. Creation alone does not prove arbitrary trimmed import. |
| Trim visible Arrangement edges without moving start | Arrangement `start_time` and `end_time` have no documented setter. Region setters exist, but their effect on external Arrangement edges is not specified as an independent trim operation. `position` describes **loop position**, not timeline placement, and its access tags say get/observe although prose says “Get and set.” [Clip][clip] | No inspected explicit Python trim implementation establishes the required outer-edge behavior. Generic setters are not proof of writable bindings. [Python interface][py-interface] | **High documentation limit / U feasibility.** Critical gate: set nonzero source start plus shorter visible duration while preserving desired arrangement start. Do not use `position` as a timeline move. |
| Move or recreate | `Track.duplicate_clip_to_arrangement(clip, destination_time)` places a Session or Arrangement clip at a destination **in beats**; `delete_clip(clip)` deletes. No direct Arrangement move function appears in the inspected Track/Clip function lists. [Track][track], [Clip][clip] | **P:** explicit duplicate/delete calls in adapter; mirrored controller code duplicates an Arrangement clip at `clip.end_time` and catches `RuntimeError`. [Python Track][py-track], [mirrored actions][mirror-actions] | **High D / medium P.** Duplicate-verify-delete is an **I candidate**, not an atomic move or identity-preserving move. Test cross-track behavior, retained bounds, collisions, errors, and undo before deleting anything. |
| Adjacent clips / collision safety | Creation has no length argument; duplication documents destination, not collision/overwrite guarantees. No no-overwrite or transactional-placement promise was found in the inspected entries. [Track][track] | A duplication call at `clip.end_time` exists, but is not a test of inserting many shortened file references next to each other. [mirrored actions][mirror-actions] | **U.** Test adjacency, both insertion orders, and temporary full-file extents. Later trimming cannot be assumed to restore neighbors damaged during creation. |
| Save/reopen | Saving a Set saves clips, positions, and settings; moving/deleting source files can break references. Collect All and Save copies referenced media. `Song.file_path` is empty until saved. [Live concepts §3.21][concepts], [Song][song] | No inspected source proves programmatic save/reopen for this workflow. Current Song function list does not document a save function. [Song][song] | **High D for application saving / U for script persistence.** Manual save/reopen is a valid experiment; do not invent a Python save API. Compare actual source segment as well as values after reopening. |
| Undo / redo | `Song.can_undo`, `can_redo`, `undo()`, `redo()` are documented. Application undo history is refreshed on opening and **not saved with a closed Set**. [Song][song], [Managing Sets §5.4.2][files] | **P, unofficial mirror:** `begin_undo_step()` / `end_undo_step()` calls exist; this is not a rollback guarantee. [mirrored undo handler][mirror-undo] | **High D / medium grouping P / U recovery.** Test grouping across deferred operations, partial failure, and redo. A `finally` ending a group does not reverse changes. |
| Failure/crash recovery | Ableton describes manual recovery using crash/undo files, requiring the **same Live version**; recovery is qualified (“can often be recovered”), not guaranteed. [Recovery article][recovery] | No inspected source demonstrates atomic import or recovery of an interrupted import. | **High D / U bridge recovery.** Use saved disposable copies and abandon unsaved partial imports if undo fails. Recovery files are not a bridge transaction log. |
| Persistent identity after edits | Max IDs identify instances, usually survive moves, but exceptions include delete/create moves. IDs are **not stored**; `LiveAPI.id` must not be reused across Max for Live runs. No durable clip UUID/custom-data property appears in the inspected Clip LOM. [API overview][overview], [LiveAPI][js-api], [Clip][clip] | Adapter IDs come from `_live_ptr`, falling back to Python `id(obj)`, stored in a process-side dictionary. Track/Song have adapter `get_data` support; that does **not** establish Clip metadata support or persistence. [Python interface][py-interface], [Python Track][py-track], [Python Song][py-song] | **High D against durable Max IDs / U Python durable identity.** Do not assert `_live_ptr` persists, or equate Max IDs and Python pointers. Test each edit case below. |

### Timing and source-coordinate traps

1. **Arrangement time is not source time.** `start_time` is song beats; unwarped `loop_start`/`loop_end` are file-relative seconds. The API overview defines beats as quarter notes and gives `seconds = beats × 60 / tempo`. Applying that conversion to a whole arrangement is an **inference restricted to constant tempo and a shared origin**, not a tempo-map algorithm. [Clip][clip], [API datatypes][overview]
2. **Markers are not automatically a complete offset model.** LOM describes `start_marker`/`end_marker` in beats, while the Live 12 Clip View manual describes unwarped UI start/end fields in minutes-seconds-milliseconds. This is an evidence gap about the API's unwarped marker behavior, not license to silently convert either way. Explicit seconds for unwarped `loop_start`/`loop_end` are stronger evidence, but their mapping to the audible Arrangement left edge still needs measurement. [Clip][clip], [Clip View §8.2.1][clip-view]
3. **A crop is not a safe trim substitute.** `Clip.crop()` says it removes material outside the region; the application manual's Crop Clip Sample makes a new sample under `Samples/Processed/Crop`. The precise Python method/UI correspondence was not tested. Do not use cropping or consolidation as evidence for preserving original media references. [Clip.crop][clip], [Clip View §8.11][clip-view]
4. **No precision result exists.** Numeric type descriptions and successful property read-back cannot prove one-sample audible alignment. The repository's one-sample tolerance remains an acceptance target, not an observed result. [API datatypes][overview]; project target: `docs/PROJECT_SCOPE.md`, Acceptance checks.

## Identity: what is and is not established

| Edit | Established evidence | Outstanding identity experiment |
| --- | --- | --- |
| Move in time / move to another track | Max IDs usually follow an object, except moves implemented as delete/create. No universal preservation promise. [API overview][overview] | Compare old/new Python wrappers, any exposed host identifier, and bridge association for time moves and cross-track moves separately. |
| Rename clip / track; reorder tracks | Names are writable; canonical paths use track/clip indices. Paths can address a different object after edits. [Clip][clip], [Track][track], [live.path][path] | Observe same-session identity, then save/reopen; do not use label or index as a key. |
| Trim / source slip | Region properties exist, but outer-edge/source behavior remains gated. [Clip][clip] | Verify association survives independent left trim, right trim, and source change without relying on unchanged geometry. |
| Split | UI Split creates independently editable pieces. No clip lineage/descendant-ID contract appears in the inspected LOM. [Arrangement §6.12][arrangement], [Clip][clip] | Record both descendants and whether one retains the original association. Decide lineage later; do not guess it from names or source overlap. |
| Duplicate | Duplication is documented; no persistent-ID allocation contract is stated. [Track][track] | Duplicate then move/rename both copies, including two equal-looking clips; check for duplicated metadata tokens and ambiguous associations. |
| Save, close, restart Live, reopen | Documented Max IDs are not stored. The adapter's pointer/cache IDs supply no durable identity guarantee. [API overview][overview], [Python interface][py-interface] | Rebuild association without runtime IDs. Inspect whether a candidate metadata channel exists on **Clip**, then test its save, split, duplication, and undo behavior. |
| Delete, undo, redo | Delete and undo/redo exist; restoration identity is unspecified in inspected docs. [Track][track], [Song][song] | Check stale references and whether restored clips keep, copy, or lose a candidate bridge association. |

**Inference:** names with embedded tokens and a geometry/path sidecar are not solved identity. Names can be edited; duplication may copy a token; geometry changes; two clips may reference the same source segment. A sidecar stores information but does not prove which reopened clip it belongs to. Ambiguity must remain visible. Nothing here selects an identity design or authorizes deletion propagation.

## Evidence-backed alternatives, not a route choice

1. **Max for Live adapter:** use documented `live.path` / `live.object`, or JavaScript `LiveAPI`, rather than assuming Python parity. These access paths are explicitly documented; Max API messages are deferred, and changes from notification callbacks require deferral. Requires Suite or Standard plus the add-on. It still has the same read-only-edge, adjacency, and ID-persistence gaps; switching language does not prove them away. [API overview][overview], [live.path][path], [LiveAPI][js-api], [Max setup][max-live]
2. **Session staging, then Arrangement duplication:** `ClipSlot.create_audio_clip(path)` creates a file-referencing Session clip; region controls can be investigated before `Track.duplicate_clip_to_arrangement` places it at a beat. **Inference:** staging may avoid inserting an initially full-file-length Arrangement clip beside a neighbor. This needs tests for trimmed bounds, offset retention, auto-warp, cleanup, and collisions. It is an internal staging strategy, not a proposal to expand the transfer contract to Session performance. [ClipSlot][slot], [Clip][clip], [Track][track]
3. **Human-assisted Arrangement edits as a fallback experiment:** Live documents dragging clips between positions/tracks, resizing edges, and splitting. These are a documented UI route to compare against script limitations, not evidence of a reliable GUI-automation API. [Arrangement §§6.7, 6.12][arrangement]

Direct `.als` rewriting is **not established as a supported alternative by the inspected primary sources**. No supported project-file identity/editing contract was found in this investigation. It would need separate format, compatibility, and data-loss research; no such implementation is proposed here.

## Minimal real-host validation procedure

This is a proposed experiment, **not an executed script**. Run only on disposable Sets and fixture-media copies. Keep an untouched saved baseline outside the experiment. Do not install third-party bridges to infer feasibility from their presence.

### 1. Record host and establish actual Python exposure

- Record full Live version/build, edition, OS, embedded Python `sys.version`, script source commit, fixture sample rate/frame count, tempo, audio preferences, auto-warp defaults, and relevant fades/processing settings. Never substitute the interpreter of an external shell.
- Install a minimal disposable Python Remote Script under the actual User Library's `Remote Scripts` folder and select it as a Control Surface, following the official article. Whether this file-only probe needs MIDI ports is a test, not established by the controller instructions. No Max license inference. [installation][remote]
- In a Live-managed script callback, obtain the song/track/clip objects and log `type`, `dir`, readable properties, relevant method `__doc__`, and exceptions. Check actual exposure of `arrangement_clips`, `file_path`, `start_time`, `end_time`, `loop_start`, `loop_end`, `start_marker`, `end_marker`, `warping`, `looping`, `sample_length`, `sample_rate`, create/duplicate/delete, undo/redo, and grouping methods. Do not call the API from an invented socket/background-thread context. Python access/thread scheduling must be validated rather than copied from Max's scheduling rules. [Python adapter call patterns][py-track], [Max thread rule, not a Python guarantee][path]
- If unavailable, record the exact missing member/error and stop that route's dependent mutation test. An optional Max probe may distinguish a Python-binding gap from a LOM/host gap; it is not a substitute Python pass.

### 2. Build one reproducible fixture and export baseline

Suggested fixture: a **12-second, 48,000 Hz PCM WAV** containing distinguishable impulses/events at known frame indices. These are proposed inputs, not discovered host settings. Keep its original hash and use a working copy. Set one constant **120 BPM**, 4/4, no tempo automation; make two ordinary audio tracks without devices.

Use the UI to create unwarped, unlooped, untransposed clips with known nonzero source offsets. Include two adjacent clips, a gap, a repeated reference to the same recording, and a non-grid placement. Save a baseline Set. Log all API fields and actual audible segment origins, not just names/counts. Do not silently discard takes, missing media, frozen tracks, muted clips, warp, looping, or processing discovered during enumeration; log and exclude unsupported cases explicitly.

### 3. Prove creation, source offset, duration, and movement separately

- On an empty scratch track, call `create_audio_clip(absolute_path, position)` using the actual exposed signature. Use a distinctive position such as **beat 8.25** (4.125 seconds at 120 BPM). Confirm its actual position, return value/error behavior, initial extent, file reference, and warp state. The beat interpretation of `position` is itself part of this test. [documented method][track]
- Request warp off; wait across a host callback boundary and read back `warping == False` before writing seconds-valued bounds. Record how many callbacks/read-backs are required; do not invent a fixed delay. [deferred warp warning][clip]
- Target **source frames 108,000 through 180,000**: source offset 2.25 seconds, duration 1.5 seconds. At the proposed start beat 8.25, the expected end is **beat 11.25** at 120 BPM. Verify the expected impulses actually play. These values are arithmetic test expectations, not measured precision.
- First read all markers/bounds before mutation. Test `loop_start`/`loop_end` separately from `start_marker`/`end_marker`, respecting the start-before-end constraint. For each valid setter order, log all fields immediately and after deferred work. Check whether outer start moves, outer end changes, or content slips. Reset to baseline between variants. Do not treat a successful `setattr` as success if the wrong source segment plays. [property contracts][clip]
- If independent trim/placement fails, test Session staging and duplication as a separate candidate. Confirm duplicate position and all source/outer bounds before considering deletion of the original. Test same-track and cross-track duplication. Read-only `start_time`/`end_time` should not be worked around by undocumented pointer manipulation. [duplication][track], [Session creation][slot]

### 4. Adjacency and preflight/failure checks

- Create two **1.5-second** clips targeting adjacent ranges, with the original 12-second source longer than each target. Run left-to-right and right-to-left import order. Inspect existing neighbors **after every create, warp update, trim, duplicate, and delete**, not only at the end. Also test placement next to a pre-existing sentinel clip. Stop on any truncation, overwrite, split, or unexpected overlap.
- Compare direct creation with staging in isolation before placement. No full-file-length temporary extent may silently damage a destination neighbor. Duplicate at the same position only in a disposable Set, on the exact named build, because release notes document historical crashes. [creation/duplication contracts][track], [11 fixes][rel11], [12 fixes][rel12]
- Test missing/invalid media and unsupported/frozen/recording/non-audio destinations. Record whether Python throws, returns a value, or only logs an error, and check for partial mutation. Missing-media preflight should change nothing; exercise an intentional failure **after one successful mutation** separately to test recovery.

### 5. Save/reopen, undo, and recovery gates

- Save a new copy manually, close, restart Live, reopen, and re-enumerate. Compare paths, sample metadata, warp state, outer boundaries, source bounds, and actual playback events. Confirm fixture audio hashes are unchanged. Saving is documented; script-induced results still need this test. [saving][concepts]
- If exposed, bracket the mutation in `begin_undo_step` / `end_undo_step`; end the group even on a test exception. Inspect Undo History, then test undo/redo of both successful and deliberately interrupted operations. Explicitly check deferred warp/bounds changes and sentinel neighbors. Grouping source exists, but all-or-nothing behavior remains unproven. [mirrored grouping][mirror-undo], [undo contract][song]
- If undo does not restore baseline, abandon the unsaved experimental copy and reopen the untouched baseline. Do not rely on undo after closing a Set. Crash recovery is a separate, version-bound fallback, not this procedure's success criterion; no forced crash is required for this minimal experiment. [Undo History][files], [qualified recovery][recovery]

### 6. Identity gate for later existing-project updates

Use two initially distinguishable clips and repeatedly log before/after snapshots. Perform rename, time move, cross-track move, track rename/reorder, left/right trim, source slip, split, duplicate, delete/undo/redo, then save/close/restart/reopen. Include duplicated clips that become indistinguishable by name, file, and source range. Compare runtime IDs/pointers separately from any proposed persisted bridge token; do not conflate them.

If actual Python introspection reveals Clip `get_data`/`set_data` or another metadata channel, record its real signature and test it. Track/Song examples do not prove Clip support. A candidate must survive every supported edit and allocate distinguishable duplicates/descendants, or report ambiguous lineage. No silent geometric matching. Test repeated return import only after a candidate identity policy is selected and proven; this research does not select it. [documented ID limits][overview], [adapter identity and metadata evidence][py-interface], [Track][py-track], [Song][py-song]

### 7. Capture pass/fail evidence

For every step, retain a small transcript: host/build/edition and script version; exact call/arguments; expected units and values; immediate and deferred read-back; actual audible source frame; exceptions; neighbor state; undo state; reopened state. Mark operations separately as pass, fail, unavailable, or not run.

Measure start/end/source-offset errors against the fixture's frame grid and report the **measured** tolerance. One sample at the proposed 48,000 Hz fixture is `1/48000` second; this is the acceptance calculation, not an achieved result. API values alone cannot establish audible/sample-accurate interchange.

## Implications for later decision tickets

- **Adapter decision:** Python has real static call-site evidence, but no runtime pass here. Max is an evidence-backed alternative with explicit licensing; Session staging is a candidate sequence. No route is selected.
- **First-transfer scope:** keep fresh-destination import conditional on source-region mapping, independent placement/trim, adjacency, and recovery. Read/export feasibility and write/import feasibility are separate gates.
- **Timing/media contract:** keep source-relative seconds/frames separate from song beats. Freeze neither marker mapping nor one-sample claims before host evidence. Source validation and missing-media preflight remain necessary.
- **Existing-project identity/conflicts:** documented runtime IDs cannot serve as durable IDs. No automatic matching, split lineage, duplicate identity, idempotence, or deletion propagation is established. Keep this milestone separate from fresh import.
- **Acceptance/handoff:** require a named Live build, edition, embedded Python version, exact calls, saved fixtures, audible/frame checks, adjacency checks, and failure/undo transcript. Do not promote D/P/I evidence to R.

Open questions for later sessions: the tested host/version floor; actual Python marker units and source-offset mapping; whether staging solves edge placement; neighbor behavior and error channels; safe grouping across deferred updates; any genuine persisted Clip metadata mechanism and its split/duplicate semantics. No new tickets were created by this worker.

**Research blockers:** none to completing the primary-source investigation. **Validation blocker:** no accessible Live host in this worker environment. Closing issue #2 records completed research, not proven bidirectional host feasibility.

## Sources

All web sources below were retrieved on **2026-11-17 UTC**. Official current pages are mutable; their exact documented version is recorded above where supplied. GitHub source citations are pinned to immutable commits.

- [Cycling ’74 LOM index][lom] — declares Live 12.4.5; [Track][track], [Clip][clip], [ClipSlot][slot], [Song][song]. Official capability surface, not a Python Remote Script compatibility table.
- [Cycling ’74 Live API Overview][overview], [live.path][path], [JavaScript LiveAPI][js-api]. Official paths, ID lifetime, data types, Max scheduling, and access mechanisms.
- [Ableton Live 11 release notes][rel11], [Live 12 release notes][rel12]. Exact feature announcements, interpreter update, and reported fixes.
- [Ableton Remote Script installation][remote] ([retrieved article JSON][remote-json]); [manual recovery][recovery] ([retrieved article JSON][recovery-json]). First-party installation/support and recovery constraints.
- Live 12 manual: [Max for Live][max-live], [Arrangement View][arrangement], [Clip View][clip-view], [Live concepts][concepts], [Managing Files and Sets][files]. Application behavior, not proof of Python API access.
- Ableton.js [README][py-readme], [Track.py][py-track], [Clip.py][py-clip], [Song.py][py-song], [Interface.py][py-interface]. Independently authored adapter source, commit `63bf873c5f1063a67011c6665d892833fcd54701`. Primary evidence for its implementation only.
- Unofficial MIDI Remote Script mirror [README][mirror-readme], [pushbase/actions.py][mirror-actions], [pushbase/undo_step_handler.py][mirror-undo], commit `e83d5192f321b24eb9daab843ac49a2d95d862b1`. Decompiled corroboration only, not an official Python specification or host-test record.

[lom]: https://docs.cycling74.com/apiref/lom/
[track]: https://docs.cycling74.com/apiref/lom/track/
[clip]: https://docs.cycling74.com/apiref/lom/clip/
[slot]: https://docs.cycling74.com/apiref/lom/clipslot/
[song]: https://docs.cycling74.com/apiref/lom/song/
[overview]: https://docs.cycling74.com/userguide/m4l/live_api_overview/
[path]: https://docs.cycling74.com/reference/live.path
[js-api]: https://docs.cycling74.com/apiref/js/liveapi/
[rel11]: https://www.ableton.com/en/release-notes/live-11/
[rel12]: https://www.ableton.com/en/release-notes/live-12/
[remote]: https://help.ableton.com/hc/en-us/articles/209072009-Installing-third-party-remote-scripts
[remote-json]: https://help.ableton.com/api/v2/help_center/en-us/articles/209072009.json
[recovery]: https://help.ableton.com/hc/en-us/articles/115001878844-Recovering-a-Set-manually-after-a-crash
[recovery-json]: https://help.ableton.com/api/v2/help_center/en-us/articles/115001878844.json
[max-live]: https://www.ableton.com/en/live-manual/12/max-for-live/
[arrangement]: https://www.ableton.com/en/live-manual/12/arrangement-view/
[clip-view]: https://www.ableton.com/en/live-manual/12/clip-view/
[concepts]: https://www.ableton.com/en/live-manual/12/live-concepts/
[files]: https://www.ableton.com/en/live-manual/12/managing-files-and-sets/
[py-readme]: https://github.com/leolabs/ableton-js/blob/63bf873c5f1063a67011c6665d892833fcd54701/README.md
[py-track]: https://github.com/leolabs/ableton-js/blob/63bf873c5f1063a67011c6665d892833fcd54701/midi-script/Namespaces/Track.py
[py-clip]: https://github.com/leolabs/ableton-js/blob/63bf873c5f1063a67011c6665d892833fcd54701/midi-script/Namespaces/Clip.py
[py-song]: https://github.com/leolabs/ableton-js/blob/63bf873c5f1063a67011c6665d892833fcd54701/midi-script/Namespaces/Song.py
[py-interface]: https://github.com/leolabs/ableton-js/blob/63bf873c5f1063a67011c6665d892833fcd54701/midi-script/Namespaces/Interface.py
[mirror-readme]: https://github.com/gluon/AbletonLive12_MIDIRemoteScripts/blob/e83d5192f321b24eb9daab843ac49a2d95d862b1/README.md
[mirror-actions]: https://github.com/gluon/AbletonLive12_MIDIRemoteScripts/blob/e83d5192f321b24eb9daab843ac49a2d95d862b1/pushbase/actions.py
[mirror-undo]: https://github.com/gluon/AbletonLive12_MIDIRemoteScripts/blob/e83d5192f321b24eb9daab843ac49a2d95d862b1/pushbase/undo_step_handler.py
