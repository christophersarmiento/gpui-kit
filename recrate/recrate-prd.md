# Recrate — Product Requirements Document

**Version:** 0.2 — draft with recorded architecture decisions reconciled  
**Product:** A local-first desktop DJ library manager and player-media exporter  
**Platforms:** macOS and Windows  
**Design baseline:** `Recrate — DJ Library Manager (4).html`, 18 boards  
**Audience:** Product, design, engineering, and a fresh model producing the implementation specification and architecture

This document is self-contained. It does not require the preceding conversation. It describes desired product behavior, not an existing implementation or a claim of hardware compatibility. Later architecture discussions record Rust, SQLite/rusqlite and gpui-kit as selected; see the [decision log](recrate-decisions.md) for confirmed choices, proposed defaults and unresolved gates. Exact analysis dependencies and the licensing/distribution model remain open.

## 1. Product summary

Recrate helps a DJ import and organize local music, prepare beat grids and cue points, and export a trustworthy USB library for supported DJ players. It provides the library-management and export workflows associated with rekordbox, without becoming a performance/mixing application.

The primary journey is:

> Bring in music or an existing library → organize and prepare tracks → review changes for a specific drive → export and verify → eject → repeat safely.

The differentiator is transparency and control: users can see what will change, what will be omitted, which files are missing, what has actually been verified, and how to recover an interrupted operation. Protecting existing music and preparation work is more important than silently completing an export.

### 1.1 Primary user

An individual DJ managing a substantial local music collection on a Mac or Windows PC, preparing playlists at home and playing them on standalone equipment. The user may have:

- Music spread across internal storage and external volumes.
- An existing rekordbox library and previously exported USB drives.
- Multiple drives and different player models or firmware versions.
- Valuable manually corrected grids, cues, loops, tags, and playlist order.

V1 assumes one active local library per running instance, not a collaborative or automatically synchronized multi-computer library. Multiple removable drives are supported.

### 1.2 Goals

1. Make routine collection and playlist management fast and understandable.
2. Preserve imported and manually prepared DJ metadata.
3. Analyze new music independently and produce player-compatible preparation data.
4. Export both legacy Device Library and Device Library Plus, separately or together, for explicitly qualified targets.
5. Safely coexist with recognized existing device libraries, with explicit replacement as a separate operation.
6. Make partial completion, external changes, and recovery visible rather than hiding them behind a success indicator.
7. Support the same core workflows on macOS and Windows, offline.

## 2. Confirmed decisions and document authority

### 2.1 Explicitly confirmed by the product owner

| Decision | Requirement |
|---|---|
| Initial platforms | **macOS and Windows.** Neither is a post-v1 port. |
| Release scope | **Full revised design.** Engineering may use staged milestones, but a one-format exporter or personal prototype is not the completed v1. |
| Analysis fidelity | **Compatible output, independent analysis.** Accurate/useful analysis and supported-player behavior matter; numerical or pixel-for-pixel rekordbox reproduction is not required. |
| Device synchronization | **One-way export initially.** Detect external changes and preserve foreign data, but do not automatically import or merge player/device edits back into the local library. |

### 2.2 Later confirmed decisions

The [decision log](recrate-decisions.md#2-confirmed-decisions) consolidates the later approvals recorded in the library, analysis and export architectures. These include the Rust/SQLite/gpui-kit stack; 100,000-track/1,000,000-occurrence scale; portable catalog/host-state separation; rational preparation time; sequential cloud handoff with managed/referenced audio; Traktor-inspired analysis/waveforms with three-band display; and staged device-database rebuilds with an isolated Plus helper. Exact versions, algorithms and compatibility combinations are not thereby qualified.

### 2.3 How to interpret this PRD

- **MUST** denotes a release requirement in this draft. **SHOULD** denotes a recommended default unless a documented trade-off is approved.
- The original four decisions and later confirmed decisions indexed in the decision log are fixed inputs for the next specification. Requirements derived from the design elaborate their behavior; they are not claims about what the prototype already implements. The log's authority rules distinguish later explicit approvals from unapproved architectural proposals.
- Product defaults proposed in §3 and performance budgets in §10 remain draft choices unless explicitly recorded as confirmed in the decision log. The scale target was subsequently confirmed; its performance has not been measured.
- Safety requirements take precedence over optimistic or abbreviated mockup copy. Mock data, file paths, device names, counts, timers, and model examples are illustrative.
- If the design implies an unsupported hardware capability, the product must disclose or block that combination, not invent support.
- The next model must expose unresolved choices and research dependencies rather than silently reduce scope or turn assumptions into facts.

## 3. Scope, defaults, and non-goals

### 3.1 In v1

- Local collection management, metadata and artwork editing, search/filter/sort.
- Ordinary playlists and folders, split-view curation, smart playlists, categorized tags.
- Reviewed file import, watched-folder review, missing-file relinking.
- Independent BPM/key/grid/waveform analysis and manual preparation.
- Single-track audio audition, hot cues, memory cues, loops, and grid editing.
- Reviewed migration from supported rekordbox local-library versions, rekordbox XML, and supported exported USB libraries.
- Device setup, legacy/Plus/dual-format export, plain-folder/M3U export, optional compatible audio conversion.
- Preservation of recognized foreign device data, explicit replacement planning, verification, recovery, and export history.
- Local-library backups, device-resource backups, restore, and preferences.
- Sequential computer handoff through immutable complete snapshots and a local working database. Include managed audio by default with verified on-demand hydration and Download all; also support referenced cloud music. Deduplicate shared cloud objects but retain them in v1 without automatic remote deletion. See [library architecture §12](recrate-library-architecture.md#12-cloud-handoff-with-managed-and-referenced-music); provider support requires qualification.

### 3.2 Proposed defaults

| Area | Draft default |
|---|---|
| Account/network | No account or internet connection required for core work. Update checks are optional. |
| Import | Offer managed-copy and reference-in-place modes explicitly; remember the user's choice. Never move or delete originals during import. |
| Metadata writes | Default Save changes only Recrate. Writing tags to audio requires an explicit action or a previously chosen write-through policy. |
| Analysis | Offer analysis after import; protect manual and imported preparation by default. |
| Existing USB library | Preserve recognized foreign data by default. Unknown or unpreservable structures block mutation. |
| Export review | Full preflight on every export. Users may skip opening the detailed diff only when no unresolved decision or destructive change requires review. |
| Backups/history | Daily local backup with a configurable retention policy; 14 recent local snapshots and 90 days of ordinary export history are initial defaults. Never expire the only recovery information for an unresolved operation. |
| Audio baseline | MP3, FLAC, PCM WAV, and PCM AIFF. Exact encoding/rate/depth combinations and tag-write support must be declared. Other codecs need an explicit scope decision. |
| Interface | English release UI with Unicode metadata and localization-ready text. Dark presentation is the design baseline; retain the design's light/system appearance option. Specific additional translations require a release decision. |

### 3.3 Out of scope for v1

- Live mixing, multiple performance decks, sync/beatmatching, crossfader, effects, controller/HID performance mapping, sampler, recording, lighting, or stems.
- Live cloud/shared-library synchronization, collaborative/concurrent-edit merging, streaming catalogs, subscriptions required for local work, mobile apps, and network player-link protocols. The explicitly supported sequential snapshot-handoff workflow above is not live synchronization; active SQLite/WAL files must not be synchronized.
- Automatic bidirectional USB synchronization or automatic merging of player-edited cues, grids, playlists, histories, or play counts.
- Recreations of proprietary analysis algorithms solely to match their numerical output.
- Universal support for every player, firmware, filesystem, source database version, or audio codec.
- Direct editing of the user's original rekordbox desktop database.
- Automatic drive formatting or filesystem repair. Show an actionable unsupported-filesystem state instead.

One-time, explicitly reviewed migration **from** a USB is in scope. It is distinct from round-trip synchronization: reconnecting an exported drive does not silently import its changes. Imported play counts may support smart-playlist rules; v1 does not promise that playing on hardware updates them in Recrate.

## 4. Important product concepts

These are semantic distinctions the architecture must preserve, not a prescribed database schema.

| Concept | Meaning |
|---|---|
| Local track | A stable Recrate identity with metadata and preparation, independent of its current path. |
| Media asset | The actual audio file, its location, availability, format, and content identity. Multiple paths or an export conversion do not automatically imply the same asset. |
| Analysis/preparation | Grids, keys, waveforms, cues and loops; includes source, version, user edits and protection state. |
| Playlist membership | An ordered occurrence of a track. A track may occur in many playlists and, by explicit choice, more than once in one playlist. |
| Smart playlist | Rules plus selection/order policy. The device receives an evaluated ordinary playlist, not a rules engine. |
| Physical device | A removable volume with a stable-enough identity to distinguish it from another drive with the same name. A mount path or drive letter alone is not identity. |
| Device library | One logical database representation on a physical device. Legacy and Plus may coexist on the same drive. |
| Exported track | A device-side identity and associated media, artwork, analysis, and references, mapped back to its source. |
| Foreign data | Device records/resources not managed by this Recrate library, including preexisting rekordbox data and externally changed resources. |
| Export plan | The reviewed snapshot of additions, updates, omissions, conversions, and removals for a specific target and source state. |
| Operation record | Durable information about intended and completed work, created/reused resources, verification, backups and recovery. A UI progress counter is not this record. |
| Capability profile | Verified constraints for player model/firmware, filesystem, library representation, codecs and preparation features. |

Deleting a playlist, removing a local track record, deleting source audio, replacing a device database, and pruning exported media are different actions. The UI must not conflate them.

## 5. Screen and navigation inventory

The revision-four artifact contains 18 boards. Two are variants of Device History, and split view is a Library mode; these are not 18 top-level navigation items.

| # | Board / surface | Required purpose and flow |
|---|---|---|
| 1 | Library | Collection/recent/needs-analysis/missing-file views; search, filters, playlists, selected-track details, waveform preview, import, analysis and settings entry points. |
| 2 | Cue editor | Single-track audition, overview/detail waveform, grid tools, hot/memory cues, loops, inspector, export-subset selection, save/discard. |
| 3 | Export to USB | Per-drive selections, capacity, effective format(s), include/conversion settings, review, progress, results and History. |
| 4 | Device history and repair | Persistent operation list and reports; interrupted copy, check, resume, discard, backups and restore. |
| 5 | Edit track info | Metadata/artwork changes, unsaved changes, library-only Save versus file write-back. Extend this surface for bulk editing. |
| 6 | Tags | Category/tag management, track assignment and matching-track filtering. |
| 7 | Library — split view | Source collection/playlist beside a destination playlist; add/reorder tracks without losing browsing context. |
| 8 | History — check found changes | Read-only findings for unexpected external changes; disable unsafe resume/discard; save report and inspect. |
| 9 | History — recovery needed | Disconnection during publication; reconnect, inspect, restore, verify and communicate unresolved recovery. |
| 10 | Import music | Sources, scan summary, duplicates/moved/unsupported decisions, copy/reference options, progress and results. |
| 11 | Analysis queue and settings | Job status, pause/resume/cancel/retry, reanalysis choices, locks, tempo/key/protection defaults. |
| 12 | Missing files | Proposed/manual matches, confidence and provenance, batch relink, unresolved results and safe record removal. |
| 13 | Device setup | Detected volume/libraries, actual supported-player profiles, effective output formats, preservation/replacement policy. |
| 14 | Export review | Per-track/resource changes and reasons, capacity, removals, omitted cues/features, required confirmations. |
| 15 | Export results | Accurate outcome counts, verification scope, problems, retry/relink, log export and eject status. |
| 16 | Smart playlist rules | AND/OR groups, typed conditions, live preview, result limit/order, save and materialize-as-normal-playlist. |
| 17 | Settings and backup | General appearance/notation, storage/watch defaults, tag policy, local backup/restore; links to feature-owned settings. |
| 18 | Import from rekordbox | Source → capability/mapping review → conflict decisions → import → outcomes and direct relink. |

Required navigation behavior:

- Library Import opens either file import or rekordbox migration. Analyze opens the queue. Missing files is accessible without a connected device.
- Device setup changes the formats displayed on Export, Review, Results, and History consistently.
- Review → Choose cues opens the affected track, then returns to the pending export with decisions retained and revalidated.
- Closing a progress dialog does not silently cancel a background task. Cancel is an explicit action; reopening shows its current state.
- Modal dialogs return focus and context to their origin. Back/Cancel/Close must distinguish navigation, abandoning edits, and stopping work.
- Library empty/first-import, no search results, no drive, read-only drive, permissions denied, unsupported format, insufficient space, and unavailable backup are required states of existing surfaces, not new primary screens.

## 6. Functional requirements

### 6.1 Collection and file import

- **LIB-01 — Import review.** Import individual files, folders recursively, and drag-and-drop selections. Scan without modifying sources; summarize new, existing, potentially duplicate, moved, unsupported and unreadable files before committing. Show per-file reasons, progress, cancellation and results.
- **LIB-02 — Managed versus referenced media.** Support copying into a user-selected managed music location or leaving audio in place. Preserve original files. Managed-copy naming collisions must never overwrite unrelated media. Optional Artist/Album organization applies to copies, not originals.
- **LIB-03 — Identity and duplicates.** Reimporting an unchanged file must not unintentionally create a second local record. Separate an exact existing asset, a relocated asset, byte-identical duplicates, and merely similar metadata. Similar titles are not proof of identity. Allow skip/import-another/reassociate decisions; any 'replace older copy' operation must disclose whether it changes a record, a reference, preparation, or actual media.
- **LIB-04 — Watch folders.** Detect new candidates and put them into import review. Do not silently ingest, delete, or relocate content. Show offline/unreadable watch locations distinctly from an empty folder.
- **LIB-05 — Removal.** Remove a playlist occurrence independently of the collection track. Removing a collection record discloses affected memberships and preparation. Deleting source audio is a separate, explicitly confirmed action and must never be implied by record removal.
- **LIB-06 — Durable outcomes.** A partially completed import exposes which tracks succeeded and which failed. Retry must not duplicate successful imports. Adding successful imports to an ordinary or newly created playlist preserves the chosen ordering policy.

### 6.2 Metadata, artwork, search and browsing

- **META-01 — Track information.** Support title, artist, album, album artist, genre, label, year/release information, track/disc number, comment, rating, color, BPM and key. Preserve imported remixer/composer/original-artist/ISRC information when present even if it is displayed under advanced details. Show path, duration, codec/container, bitrate, sample rate, bit depth and availability as technical information.
- **META-02 — Editing.** Provide single-track and bulk edits with mixed-value handling, field validation, unsaved-change indicators and undo for supported local edits. Bulk editing changes only explicitly selected fields. Changing a displayed key notation does not redetect or alter the musical key.
- **META-03 — File tag policy.** Library-only, ask-per-save and explicit write-through policies are supported. Report local-save success separately from audio-tag-write success. Unsupported/read-only media retain a failed or pending write with a reason; do not show a false successful write. Preserve unrelated embedded tags and audio content. Offer original-tag restoration where a verified tag backup exists.
- **META-04 — External edits.** Reread tags with a reviewable precedence/conflict policy. Do not silently replace manually prepared grids, cues or key overrides. Detect underlying audio-content changes separately from tag-only changes and mark dependent analysis stale where appropriate.
- **META-05 — Artwork.** View, add, replace and remove artwork; accept supported external images and embedded art. Track local and source-file changes separately. Generate export derivatives as required by the profile; missing art is a warning unless the target requires it.
- **BROWSE-01 — Browsing.** Search across relevant textual metadata and filter by BPM/key/rating/color/tags/date/analysis/availability. Provide stable sorting, configurable columns, multi-selection and keyboard navigation. Clearly distinguish the current sorted view from stored playlist order.
- **BROWSE-02 — Search behavior.** Support Unicode and an explicitly documented normalization/case policy. Do not require reproduction of rekordbox's historical collation implementation. Numeric range filtering is different from textual substring search.

### 6.3 Playlists and tags

- **ORG-01 — Ordinary organization.** Create, rename, move, duplicate and delete playlists/folders; reorder tracks and siblings; prevent hierarchy cycles. Confirm destructive scope. Playlist duplicates are an explicit user choice, not a global track-identity rule.
- **ORG-02 — Split view.** Browse a source collection or playlist beside the destination; add selections by button/drag-and-drop, indicate already-present tracks and offer undo. Preserve source filters and destination order.
- **ORG-03 — Smart playlists.** Support nested AND/OR groups and typed conditions for at least BPM, key, tags, date added, rating and play count, as shown in the design. Validate ranges/keys, define missing-value behavior, preview matches, choose limit and deterministic sort/tie-break policy, and save a snapshot as an ordinary playlist. Empty results and evaluation errors must be distinguishable.
- **ORG-04 — Export materialization.** Freeze smart-playlist membership and order into the reviewed export plan. Do not reevaluate silently mid-export. A later result change appears as a new export diff.
- **ORG-05 — Tags.** Create/rename/reorder/remove categories and tags, assign to single or multiple tracks and filter matching tracks. Within-category ANY and across-category ALL matching is the design default. Removing a tag discloses affected assignments. Export mappings to My Tag are profile-specific; explain category/length/count omissions rather than silently taking arbitrary categories.

### 6.4 Missing and changed media

- **HEALTH-01 — Availability.** Distinguish unavailable volume, permission failure, missing file and invalid/replaced audio. A disconnected music drive is not evidence that all its files should be removed.
- **HEALTH-02 — Relink review.** Search selected folders and propose old → new matches with the evidence used. Ambiguous/weak matches require user review; offer manual Locate. Preserve track identity, playlist membership and preparation when reassociating the same audio asset.
- **HEALTH-03 — Different audio.** Do not apply old cue/grid positions to a different edit or duration without warning and explicit resolution. Relinking updates dependent paths and reports partial failures; it is not success merely because a database path changed.
- **HEALTH-04 — Follow-up.** Migration, import and export problems link directly to collection-wide Missing files. Removing an unresolved record does not delete a file elsewhere or force deletion from an offline export.

### 6.5 Analysis and track preparation

- **AN-01 — Independent analysis.** Analyze supported media for BPM, beat positions/downbeats, musical key, and overview/detail waveforms. Support constant-tempo and variable-tempo material. Use independent algorithms; no requirement to reproduce rekordbox's thresholds, smoothing, detector internals or exact bytes from identical audio.
- **AN-02 — Queue.** Queue multiple tracks; show queued/running/paused/complete/failed/protected/cancelled states and per-track reasons. Permit pause, cancellation and retry without replacing valid prior analysis with incomplete results. Closing the queue keeps work observable from Library.
- **AN-03 — Selective reanalysis and protection.** Select BPM/grid, key and/or waveform for reanalysis. Protect manual/imported grids and overrides by default. Require explicit consent to replace protected values; retain a recoverable prior state. Ordinary analysis does not erase manually created cues or loops.
- **AN-04 — Provenance.** Distinguish imported, automatically generated and manually edited preparation. Record enough analyzer/version/settings provenance to decide whether regeneration is needed. A changed display preference alone must not trigger analysis.
- **PREP-01 — Audition.** Provide single-track play/pause, seek, output volume, waveform navigation, zoom and beat-based navigation, using an available OS audio output. This is a preparation tool, not a mixing deck.
- **PREP-02 — Grid editing.** Display bars/beats and permit BPM entry, tap tempo, downbeat placement, grid alignment/nudge, correction of tempo-multiple errors, and lock/unlock. Variable-tempo data must not be flattened by unrelated edits; the spec must define suitable section/anchor editing. Grid changes do not silently move absolute cue positions.
- **PREP-03 — Cues and loops.** Create/edit/delete named and colored hot cues, memory cues, memory/hot loops and active-loop intent; set/adjust positions and loop bounds, optionally quantize to the grid, copy/assign between supported cue types, and save/discard/undo local edits. The design's A–H pads are the default UI, not a universal hardware limit.
- **PREP-04 — Local richness versus export limits.** Preserve all local/imported preparation even when a device cannot represent it. For the legacy ten-memory-cue export path, let users select up to ten. If over capacity and unselected, require either an explicit subset or a reviewed 'use first ten' acknowledgement. Define the ordering of 'first' visibly. Never silently truncate local data.
- **PREP-05 — Other target limits.** Hot-cue slots, loop support, comments/colors, My Tag mappings and waveform modes follow the qualified profile. Do not infer unlimited memory cues on Plus from its database schema. Distinguish what can be serialized from what the player demonstrably exposes.
- **PREP-06 — Timing fidelity.** Maintain a canonical audio timeline and preserve imported/edit precision. Convert to each target's units with a declared quantization/codec-alignment budget. Local save/reload must not accumulate timing drift; export round trips must remain within the profile's tested budget.

### 6.6 Migration from rekordbox

- **MIG-01 — Read-only sources.** Support qualified local-library versions, rekordbox XML and qualified exported USB libraries. Read an isolated, consistent snapshot; do not modify the original database, audio, settings or device merely to inspect it. Handle a live/changing source through an explicit close/retry or safe-snapshot strategy.
- **MIG-02 — Source-specific review.** Report the actual selected source's tracks, playlists/folders, grids, keys, cues/loops, ratings/colors/comments, artwork and tag information. Do not show the desktop database's fields/counts for an XML or USB source that lacks them. Disclose unsupported features and inaccessible/missing media before commitment.
- **MIG-03 — Conflicts.** Resolve exact existing tracks separately from probable duplicates. Offer keeping Recrate preparation or replacing it with the imported version; default to preserving Recrate. A 'keep both' option must explain cue-slot conversion and which grid remains active. Do not claim two conflicting beat grids were merged. The spec must retain the unselected preparation in a recoverable import snapshot or equivalent record.
- **MIG-04 — Intelligent playlists.** Translate only supported rules with equivalent semantics. Unsupported rules may be imported as a normal playlist of known source matches, with explicit notice. Do not silently broaden the query; if source matches are unavailable, report that limitation.
- **MIG-05 — Completion.** Back up Recrate before mutation. Publish per-item/category outcomes, link to relinking, support idempotent retry and preserve provenance. Import is not ongoing synchronization. Reimporting an unchanged snapshot should not multiply records or cues.

### 6.7 Device setup and export planning

- **DEV-01 — Detection.** Identify the connected volume, writable status, filesystem, available capacity and recognized libraries. Reconnecting a drive at a new mount path must not lose its history; a different drive with the same name must not inherit authorization.
- **DEV-02 — Profiles and formats.** Offer legacy Device Library, Device Library Plus, both where qualified, and plain-folder/M3U export. Choose effective format(s) from target capabilities; 'older/newer' is explanatory UI, not the capability algorithm. Display the same effective choice throughout the flow. Dual format adds databases/necessary derivatives, not a second copy of every unchanged audio file by default.
- **DEV-03 — Honest qualification.** List tested player model, firmware, filesystem, Recrate/exporter version and tested features. Keep unsupported, untested, and supported distinct. Empty placeholder tables cannot be presented as successful tests. Recognition of a format is necessary but not sufficient evidence of safe writing.
- **DEV-04 — Existing content.** Inspect without writes. Default to preserving recognized foreign data. If the writer cannot preserve relevant records, fields, sections or references, block the merge. Replacement requires a separate exact impact review, not merely selecting a radio button.
- **EXP-01 — Selection.** Select individual tracks and/or playlists/folders, including smart playlists. Show the managed selections already on that device. Deselection and playlist removal must distinguish removing membership, removing a device track record and reclaiming a media file. Shared references across both libraries and foreign data prevent unsafe pruning.
- **EXP-02 — Diff.** Before mutation, calculate copy/reuse/update/convert/skip/remove actions, quantities and reasons, missing/stale required analysis, field/feature omissions and capacity. Include temporary working space, expanded conversions and required backups, on both the device and local storage as applicable.
- **EXP-03 — Decisions.** Missing media, unsupported encodings, lossy preparation mappings, destructive removals and replacement must be visible. Export is blocked until required decisions are resolved. Skipping a missing track requires explicit acknowledgement or a clearly selected reviewed policy; do not silently reuse an outdated copy as if it were current.
- **EXP-04 — Plan freshness.** A review belongs to a particular device and source snapshot. Reconnects or source/target changes invalidate affected checks. Revalidate before applying the plan; do not reuse a past confirmation to authorize a different deletion set. Users may continue browsing/editing; later edits are either excluded into a subsequent plan or cause an explicit replan.
- **EXP-05 — Media and analysis.** Copy audio, generate required artwork and analysis resources, encode metadata and preserve intended playlist hierarchy/order. Produce the waveform/grid/cue variants required by each qualified target, even when the UI uses a different display representation. Validate destination names, paths and references.
- **EXP-06 — Conversion.** Offer compatible export-only conversion, including the design's FLAC-to-AIFF case when actually required. Do not convert solely because a player is called 'old.' Show new size and encoding choices, preserve originals, and validate timeline/seek/analysis correspondence for the derivative.
- **EXP-07 — Plain folders.** Export audio and portable M3U playlists without claiming on-player cue/grid/waveform library support. Provide deterministic collision-safe paths and clearly describe unsupported preparation metadata.
- **EXP-08 — Incremental repeatability.** Repeated unchanged exports reuse verified resources and preserve stable mappings without duplicating tracks/playlists. Metadata, preparation, media and artwork changes must not require indiscriminately recopying everything.

### 6.8 Coexistence, publication and verification

- **SAFE-01 — Preserve foreign semantics.** Preserving a library means preserving its unrelated track metadata, playlist membership/order, cues, grids, artwork and references, not just keeping its playlist names. Shared database files may require rewriting; verify semantic preservation and retain exact pre-change resource backups. Preserve unknown content only when it can be done safely; otherwise refuse mutation.
- **SAFE-02 — Shared assets.** Reusing identical audio does not authorize overwriting shared analysis or cue data. If Recrate and foreign preparation differ, use a safely separate supported representation or block and explain the conflict. Do not edit foreign preparation in order to make Recrate's version fit.
- **SAFE-03 — Backups before mutation.** Before changing an existing device resource, capture and validate everything required to restore it: each affected library, analysis/auxiliary/artwork resource as applicable, and a record of resource creation/deletion. Capture a consistent database generation, not an arbitrary main file while its sidecars contain newer state. A required backup failure blocks mutation.
- **SAFE-04 — Durable operation record.** Persist the exact target identity, plan, owned/created/reused resources, before-state, verification and backup references before relying on them for retry or recovery. Name/size/modification-time matching alone is insufficient evidence to delete a file. The spec must choose adequate identity/content verification and specify partial-file handling.
- **SAFE-05 — Recoverable publication.** Prevent planned database entries from pointing to incomplete media as a normal successful result. A crash, cancellation or disconnect must yield a discoverable operation state. Do not assume an atomic transaction spans FAT32 files, legacy databases, Plus databases and analysis resources. Safety requirements apply even if only one representation of a dual-format export was updated.
- **SAFE-06 — Verify the result.** Read back relevant exported resources and verify planned media content, database structure and relationships, paths, counts/order, preparation mappings and preservation of foreign content. A database write success or file-copy return code alone is not verified completion. Distinguish application-level verification from actual player qualification.
- **SAFE-07 — One-way authority.** External changes trigger a target diff and, where unsafe, inspection. Never silently import them into Recrate or overwrite them on reconnect. Resuming a plan cannot assume the drive still matches the earlier generation. Writing and checking the same device must be coordinated across jobs/instances.
- **SAFE-08 — Explicit destruction.** Delete only resources authorized by the current reviewed operation, positively owned/created as applicable, and unreferenced by either library at deletion time. A display marker or ownership label is not sufficient. Unrelated files and foreign data are preserved in Keep mode.
- **SAFE-09 — Replacement versus media deletion.** Replacing a device library does not prove that its audio exists on the computer. The draft safety default is to retain preexisting audio when replacing the database. Any requested media cleanup must be separately inventoried, authorized and backed up in a recoverable way; a database-only backup cannot restore deleted audio. Do not repeat the mockup's unconditional assurance that rekordbox retains another copy.
- **SAFE-10 — Cancellation and eject.** Stop at a defined safe boundary or finish a clearly disclosed critical step before stopping. Show stopping/verification/recovery states honestly. 'Safe to eject' is independent of 'all requested tracks exported'; actual eject succeeds only after the OS confirms it. Handle device-busy and eject-failure states.

### 6.9 Results, history and recovery

- **REC-01 — Results.** Provide per-track/resource copied/updated/reused/skipped/failed/omitted results and reasons. Success with user-approved omissions is distinct from full success and from an unverified/partial result. Show verification scope and both library statuses where applicable. Support retry, relink and exportable diagnostic reports.
- **REC-02 — Persistent history.** Preserve operation history after dialog close, restart or device disconnect. Reports are associated with a device identity and export generation. Historical counts remain historical until a fresh check succeeds.
- **REC-03 — Interrupted copy.** Initially show last-known progress. Require the correct device to be reconnected and checked without mutation before enabling Resume or Discard. Separate verified completed files, incomplete files, absent files and unexpected changes.
- **REC-04 — Safe resume/discard.** Resume reuses only verified completed work and resolves incomplete copies. Discard deletes only files provably created by that interrupted operation and not currently referenced by either library, rechecked immediately before deletion. Keep preexisting/reused files. If any proof is missing, stop for inspection rather than guess.
- **REC-05 — Unexpected changes.** Offer findings, save report, reveal location and recheck. Disable stale-plan resume/discard when the target differs unexpectedly. A new export requires a new plan, review and backup against the current state.
- **REC-06 — Interrupted publication.** If a drive disconnects during library/resource publication, display Recovery needed; do not claim its old library is intact. On reconnection, verify device and current state, propose safe restoration, restore affected resources from verified backups and read them back before declaring recovery successful. Unexpected later changes require inspection/confirmation rather than automatic clobbering.
- **REC-07 — Recovery failure.** Missing/corrupt backups, further disconnects, partial restore, disk-full and verification failures remain visible and recoverable where possible. Retain the operation record and known-good backups. No success claim is permitted just because a restore was attempted.
- **REC-08 — Historical restore.** Let users inspect the scope and consequences of restoring a pre-export snapshot. Capture current recoverable state first. Explain that restoring database/analysis resources does not resurrect audio deleted elsewhere. After restore, revalidate references and identify missing media before claiming the drive is usable.

### 6.10 Settings and local-library backups

- **SET-01 — Preferences.** Provide appearance, key notation, storage locations, import/watch defaults, tag policies and backup preferences. Use native macOS/Windows path display and shortcuts. Settings must affect the corresponding workflow consistently; decorative switches with no behavior are not release features.
- **SET-02 — Local backup.** Back up collection records, organization/rules/tags, preparation, settings necessary to interpret them, provenance and relevant asset references. Include or safely regenerate non-source artwork/analysis as declared. Audio is excluded by default and clearly stated. Validate backup completeness and compatibility.
- **SET-03 — Local restore.** Preview what changes will be replaced, back up the current state, restore and validate. Fail without discarding the current known-good library if the snapshot is unusable. Local restore never silently changes connected devices or source audio; subsequent export starts with a fresh target comparison.
- **SET-04 — Retention and storage.** Show location, date, scope, size and validation state of snapshots; permit manual backup and configurable automatic retention. Keep unresolved-operation backups regardless of normal expiry. Handle inaccessible backup locations and local disk-full explicitly.
- **SET-05 — Library relocation.** Moving library storage is an explicit job with progress, failure and recovery behavior, not an instantaneous path-setting change. Preserve track identities and distinguish moving Recrate's database/resources from moving managed audio or reassociating external files.

## 7. Observable export and recovery states

The implementation spec must refine these into complete state machines, including transitions on restart, cancellation and I/O failure. They are not all separate pages.

| State | User-facing contract |
|---|---|
| Inspecting / planning | No target mutation. Show source/target capabilities and the proposed diff. |
| Needs decisions / blocked | Explain exactly what must be resolved; do not permit Export through another entry point. |
| Ready / approved | The plan and acknowledgements apply to a named device and generation. |
| Backing up / copying / converting / publishing | Show stage and progress; keep the operation record durable and offer explicit safe cancellation. |
| Verifying | Files have been written but export has not yet been declared verified. |
| Complete | Planned changes and preservation checks pass, with any approved omissions disclosed. |
| Finished with problems | Report partial/skipped/failed work and whether the current drive library is verified and usable. |
| Interrupted — unchecked | Display last-known information only; require a new read-only check. |
| Checked — resumable / discardable | Enable only actions justified by the current state and resource ownership evidence. |
| Needs inspection | Unexpected changes make the old plan unsafe. Do not mutate while diagnosing. |
| Recovery needed / waiting for device | Publication or rollback is unresolved. Do not claim a working old library or successful restoration. |
| Recovering / verifying restore | Restoration in progress; a new failure stays unresolved. |
| Recovered / discarded | Explain precisely which resources changed and what was retained; verify the claimed outcome. |

A disconnect, volume change or external modification invalidates relevant prior checks. History's illustrative successful-copy scenario cannot be used to assume publication failures behave the same way.

## 8. Safety clarifications that supersede mockup shortcuts

1. A supported-format badge must reflect qualified read/write behavior, not merely recognizing a file extension or opening a database.
2. Legacy and Plus are separate representations sharing some resources; 'one shared library' in UI copy must not hide partial dual-format success.
3. A table of Plus cue rows does not prove an unlimited player cue capacity. Serialize and advertise only tested mappings.
4. 'First ten' is an explicit export choice, not a local-data deletion policy. An active loop or essential marker must not be silently dropped.
5. Recovery evidence must be stronger than filename/size/time. Those attributes are useful diagnostics, not sufficient deletion authorization.
6. Database-only restore is not full media restore. Retain existing audio or explicitly back it up before separately authorized destruction.
7. Foreign-data preservation may require declining a merge. Showing 'Keep' must not force an unsafe best-effort rewrite.
8. Automatic backup restoration is conditional on device availability and verified target state. Attempted recovery is not completed recovery.
9. Safety checks are workflow invariants, not just disabled buttons; alternate commands, retries and restarts must enforce them too.

## 9. Acceptance scenarios

The next specification must turn these into automated and manual tests and map them to the requirements above.

| ID | Scenario and expected outcome |
|---|---|
| A01 | Import supported music by copy/reference on both OSs; original bytes remain unchanged, results identify failures, and unchanged retry creates no unintended duplicate records. |
| A02 | Move a folder or reconnect it under another mount path/drive letter; relink preserves IDs, playlists and preparation. Ambiguous or different-audio matches are not silently accepted. |
| A03 | Bulk-edit one field across mixed values; other fields stay intact. A read-only file yields a successful local edit plus a clearly failed/pending tag write, not a false all-success message. |
| A04 | Add/remove/reorder ordinary playlist occurrences and use split view; stored order and duplicate policy remain stable after restart. |
| A05 | Evaluate nested smart rules with empty/missing values; preview equals the materialized export snapshot. Evaluation failure is not mistaken for an intentionally empty playlist. |
| A06 | Analyze constant and changing-tempo tracks; manual overrides survive default reanalysis. Selected unlock/reanalysis replaces only approved outputs. |
| A07 | Save/reload/edit cues and loops at multiple sample rates and codecs; local timing is stable and each supported export/import mapping stays within its declared timing budget. |
| A08 | Export a track with more than ten memory cues through the legacy path; require a selected subset or explicit first-ten acknowledgement, retain all local cues and report omissions. |
| A09 | Import each supported migration source; mappings/counts reflect that source. Missing media, unsupported rules and preparation conflicts are reviewed; source artifacts remain unchanged. |
| A10 | Export legacy, Plus and both, on macOS and Windows, to disposable test targets; independent readers validate structures and relationships and actual qualified players demonstrate browse/load/grid/cue/loop/waveform/artwork behavior. |
| A11 | Export unchanged selections repeatedly; mappings and counts remain stable, and media is not copied or duplicated unnecessarily. |
| A12 | Keep a recognized foreign library while adding Recrate playlists; unrelated foreign semantics and shared resources remain intact. An unpreservable structure blocks the write. |
| A13 | Remove a managed playlist with shared tracks; only authorized memberships/records/resources are removed, with references checked across both formats and foreign data. |
| A14 | Disconnect during copy, restart Recrate, reconnect the same drive under another path; display last-known state, check, then safely resume or discard only proven new unreferenced files. A same-named different drive is rejected. |
| A15 | Change the target externally after preflight or after an interrupted export; old actions are invalidated and inspection/replanning is required. No device edits are automatically merged into Recrate. |
| A16 | Disconnect/crash/fail after either database or an analysis resource is published; show unresolved recovery, retain backups, restore when safe, and verify before declaring success. Include repeated failure during restoration. |
| A17 | Fail backup creation, conversion, short write, final flush, read-back, or reference validation; no false Complete state. Verify the recorded current drive state and recovery options. |
| A18 | Restore local/device snapshots; scope is explicit, current state is protected, missing audio is reported, and a local restore does not mutate connected devices. |
| A19 | Cancel and close/reopen each long-running workflow; jobs neither disappear nor restart unintentionally. Eject is blocked/failed appropriately while the OS or a job holds the drive. |
| A20 | Exercise empty/no-results/no-device/offline/unsupported/permission/disk-full states with keyboard and assistive technology; users can understand and recover without relying on color alone. |

## 10. Nonfunctional requirements and proposed targets

### 10.1 Hard requirements

- **Offline and private:** Core management, audition, analysis, migration, export and recovery work without a service. No upload of music, metadata, paths or diagnostics without explicit permission. Logs are local by default; shared reports offer path/metadata redaction.
- **Cross-platform correctness:** Handle Unicode, filename collisions, case-sensitivity differences, Windows reserved names/drive letters, macOS permissions/mount changes and external-volume availability. Preserve IDs independent of platform-specific paths.
- **Security:** Treat media, tags, artwork, XML, analysis files and device databases as untrusted input. Bound parsing and allocation, reject path traversal/out-of-volume writes, and constrain inspection/repair side effects. Do not include recovered secrets in reports or this PRD.
- **Durability:** Survive application restarts and injected I/O failures without claiming unverified success. Real hardware failure can exceed the app's control; communicate uncertainty and preserve recoverable evidence instead of guaranteeing impossible cross-file atomicity.
- **Accessibility:** Keyboard-operable primary workflows, logical focus/labels, non-color status cues, platform accessibility APIs, scalable text and usable layouts at smaller window sizes than the mockup's fixed canvas.
- **Operational consistency:** Background work must not freeze browsing. Define concurrency/locking so analysis, tag writes, local restore and export do not race over the same resources. Reopening the app restores visibility of unresolved work.
- **Distribution:** Provide installable, appropriately signed desktop builds and a documented update/rollback strategy that protects library schema compatibility. Resolve component/model/codec licensing before shipping.

### 10.2 Draft performance and analysis-quality budgets

The scale below was subsequently confirmed in the library architecture; the remaining quantitative targets are proposals for specification/benchmarking, not measured results or user-approved guarantees. The spec must name reference hardware, dataset shape, cold/warm conditions and how each metric is measured.

| Area | Proposed target |
|---|---|
| Scale (confirmed design target) | Interactive management of 100,000 tracks and 1,000,000 playlist occurrences on a representative 16 GB RAM desktop/laptop; performance remains to be measured. |
| Warm search/filter | p95 first useful results within 250 ms for ordinary queries at that scale. Expensive smart rules show cancellable progress rather than blocking. |
| Interaction | Common selection/keyboard feedback within 100 ms; smooth virtualized browsing while background jobs run. |
| Startup | Usable library shell within 5 seconds on the agreed reference SSD system; device scanning/analysis can continue separately. |
| Preparation throughput | Sustained full baseline analysis faster than realtime on the reference system, with reported throughput rather than fabricated ETAs. |
| Export performance | Avoid full media recopy on unchanged exports; measure copying, conversion, database work and verification separately. Device I/O is a declared variable, not a fixed speed promise. |

For analysis quality, establish a representative annotated corpus covering electronic constant-tempo music, variable tempo, silence/intros, short/long files, clipping and supported codecs. Measure tempo accuracy including half/double errors, beat/downbeat alignment and drift, key classification with ambiguous cases, and waveform temporal alignment. **The numerical pass thresholds and selected implementation are open engineering/product decisions that must be approved before analysis is considered release-ready.** Agreement with rekordbox is not the ground truth or the acceptance metric.

## 11. Delivery milestones and release gates

All milestones contribute to the full v1; they are not permission to ship a reduced scope under the full-v1 label.

1. **Qualification foundation:** Freeze versioned fixtures, define capability profiles and an independent validation harness, select initial real hardware and OS test targets, evaluate licensing.
2. **Local library:** Stable identities, metadata, organization, search, imports, relink, backups and read-only migration prototypes with deterministic tests.
3. **Preparation:** Independent analysis selection/benchmarks, audition, grid/cue/loop editing, protection and timing round trips.
4. **Export on disposable targets:** Separate legacy/Plus writers, required shared resources, reviewed plans, incremental media handling, conversion and plain exports. No unsupervised writes to valuable devices.
5. **Trust and recovery:** Coexistence, source/target change detection, durable history, verification, injection of failures across copy/publication/restore, safe discard and replacement semantics.
6. **Integrated desktop release:** All core design workflows on macOS and Windows, accessibility/performance budgets, actual player testing and published support matrix.

**V1 release gate:** The approved supported combinations for both library formats pass the required scenarios, the UI makes unsupported combinations explicit, recovery is tested, and no unresolved issue can silently delete foreign/source data or report an unverified export as successful. Exact initial player/firmware and OS versions must be selected before qualification; the mockup's examples do not select them.

## 12. Evidence available to the architecture/specification author

### 12.1 Design artifact

- Latest reviewed file: `/Users/christopher/Downloads/Recrate — DJ Library Manager (4).html`.
- SHA-256: `f01af4ca9d62f2496c8b5190d9efbcda990124f5a7c154ac8c327db77d0d36ca`.
- It is a bundled HTML design/prototype with nested compressed boards and sample interactions, not a working product. Paths, statistics and successful operations inside it are mock data.
- If the file is not available to the next model, §5 supplies the screen inventory and this PRD supplies the behavior. Do not claim to have visually inspected an inaccessible artifact.

### 12.2 Repository research

In the current handoff repository, paths below are relative to the repository root. They are optional technical evidence, not prerequisites for understanding the product:

- `research/rekordbox/README.md` — baseline and evidence index.
- `research/rekordbox/readiness.md` — research limits and outstanding qualification.
- `research/rekordbox/library-workflow-matrix.md` and `library-lifecycle.md` — library operation contracts.
- `research/rekordbox/export-lifecycle.md` and `architecture.md` — identities, resources and export boundaries.
- `research/rekordbox/legacy-device-library.md` — legacy database observations.
- `research/rekordbox/device-library-plus.md` — Plus schema/container observations.
- `research/rekordbox/waveform-storage.md`, `waveform-pipeline.md`, `waveform-audit.md` — analysis payloads and corrections to older notes.
- `research/rekordbox/beat-key-analysis.md` — analysis/preparation persistence and investigation limits.
- `research/rekordbox/evidence/final-verification-index.md` — preserved fixtures and diagnostics.

The research baseline is copied rekordbox 7.2.18.0311 ARM64 plus selected exported artifacts. Static observations and structural fixture checks **do not establish a complete compatible writer, numerical parity, or actual player qualification**. Existing diagnostics are research probes, not a production parser or a full product test suite.

Relevant established distinctions for planning:

- Legacy uses `export.pdb` / `exportExt.pdb`; Plus uses a separate keyed SQLite/SQLCipher container, `exportLibrary.db`. A successful container read is not a complete writer/compatibility qualification.
- Media, artwork and external analysis files are separate resources; some are shared across logical device databases. `.DAT`, `.EXT`, and `.2EX` are relevant analysis extensions; `.2EX` is the literal extension despite internal 'ex2' naming.
- Different targets may need different waveform families. Modern three-band overview/zoom data use PWV6/PWV7; generating only those is not proven sufficient for broad player support.
- Origin IDs, local IDs, device IDs, persisted relative paths and cached absolute paths are different representations.
- Research found multiple independent write/failure boundaries. Recrate must implement its own sound error/short-write/flush/recovery behavior, not reproduce defects observed in another application.
- The owner's independent-analysis decision supersedes any research milestone proposing exact detector reproduction as a prerequisite. File-format correctness and hardware qualification remain mandatory.

## 13. Decisions the next specification must resolve

Consult the [decision log](recrate-decisions.md) first: later architecture decisions have resolved parts of the original handoff questions. Do not reopen those choices or present unresolved details as confirmed owner requirements. Propose options with a recommendation for remaining gaps; request owner decisions only where they materially change scope or risk.

| Topic | Required resolution |
|---|---|
| OS and hardware matrix | Minimum macOS/Windows versions, CPU architectures, named player/firmware targets and test equipment. Both platforms and both device-library families remain in scope. |
| Codec/filesystem matrix | Exact decoder, tag-writer, conversion and device-playback combinations; supported partition/filesystem constraints and capacity/path limits. |
| Analysis | Licensed candidate algorithms/models, annotated corpus, objective thresholds and performance evidence. No exact rekordbox-analysis dependency. |
| Stack | Rust, SQLite/rusqlite, gpui-kit and the isolated SQLCipher helper are selected. Resolve pinned versions, native integration, audio/analysis dependencies and distribution details within those choices. |
| Identity and time | UUID identities, portable/host-state separation and rational decoded-audio coordinates are selected. Complete production source/device mapping, content-change, codec-alignment and error-budget contracts. |
| Publication/recovery | Complete staged database rebuilds are selected. Refine the durable operation protocol, live-resource publication boundaries, per-format/shared-resource coordination, ownership proofs, verification and interruption recovery. |
| Coexistence | Supported read-modify-write versions, unknown-field policy, resource sharing under differing cues/grids and refusal conditions. |
| Migration conflicts | Supported desktop/XML/USB source versions and explicit active-grid/cue behavior for 'keep both.' |
| Backups/restore and handoff | Complete recovery scope, schema compatibility, retention exceptions, local restore versus current device state, and cross-host recovery restrictions. Sequential snapshot handoff is selected; local backup excludes audio by default while default cloud handoff includes managed audio. |
| Preparation UX | Variable-tempo editing mechanics, loop precision/active-loop controls and protected output semantics. |
| Public product concerns | Distribution/licensing/pricing, telemetry policy beyond opt-in, support process and translated languages. None may make offline core functionality depend on a service without a new product decision. |

## 14. Requested output from the next model

Produce a technical specification and architecture proposal, not production code, that includes:

1. A traceable mapping from this PRD's requirements and 18 design boards to components, workflows and tests.
2. Implementation details and deployment targets within the confirmed stack, with rationale for remaining choices; do not assume the research repository is an application starter.
3. Domain model and invariants for identity, media, preparation, playlists, rules, tags, provenance, jobs, devices, export plans and backups.
4. Public/internal boundaries between library operations, media I/O, analysis, preparation UI, format readers/writers, export orchestration and OS integration.
5. Explicit state machines for import, analysis, export, inspection, restore, cancellation and recovery, including crash/restart behavior.
6. Data/file formats, migration/versioning policy, timing/quantization contracts, concurrency and consistency rules.
7. Threat/failure model, backup/verification design and proof obligations before any destructive device action.
8. Compatibility research spikes and an independent, fault-injected and hardware-backed test plan. Identify claims that cannot yet be supported.
9. Incremental engineering milestones that lead to the complete v1, with measurable gates rather than unsupported time estimates.
10. A decision log distinguishing confirmed requirements, proposed defaults, recommendations and unanswered release-blocking questions.

Do not replace this product with a web-only tool, a mixing application, a cloud service, an exact rekordbox analyzer clone or an automatic bidirectional synchronizer. Do not claim that a new writer is player-compatible because a sample database parses.
