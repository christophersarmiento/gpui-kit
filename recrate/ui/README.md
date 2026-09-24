# Recrate design handoff

Recrate is a local-first desktop DJ library manager for macOS and Windows.
It imports and organizes music, prepares beat grids and cue points, and
exports verified libraries to USB drives for supported DJ players. It is not a
performance app: there are no decks, mixer or effects.

This folder holds the interactive mockups (`design/`) and a guide to reading
them against the product requirements.

## What takes precedence

1. **The PRD, [`../docs/recrate-prd.md`](../docs/recrate-prd.md)**, is the
   source of truth for behavior, scope and safety. Requirement IDs such as
   `SAFE-09` and `EXP-03` refer to it.
2. **The decision log, [`../docs/recrate-decisions.md`](../docs/recrate-decisions.md)**,
   records later approvals and open gates, including the stack, scale and
   publication model. The architectures beside it in `../docs/`, such as
   [`recrate-library-architecture.md`](../docs/recrate-library-architecture.md)
   and the analysis and export architectures, refine it.
3. **This README** summarizes how the mockups apply the PRD. Where it
   disagrees with the PRD or the decision log, they win.
4. **The mockups** show layout, copy and interaction for the states they
   draw. They are not complete, and some copy and behavior are shortcuts. See
   [Mockup shortcuts](#mockup-shortcuts). Where a mockup disagrees with the
   PRD, the PRD wins. Report the conflict rather than copying it.

## Before you start

- Recrate is an application built on `gpui-kit`. Put application code in its
  own crate. Don't add Recrate-specific behavior to `gpui-base` or
  `gpui-component`.
- If this folder sits inside the gpui-kit repository, that repository's
  Design Guides and Coding Guides (`website/docs/design-guides.md` and
  `website/docs/coding-guides.md`) also apply. Elsewhere, follow the PRD's
  nonfunctional requirements (§10) and the interface rules below.
- Everything in the mockups is sample data: track names, counts, sizes, dates,
  paths, player models and firmware versions. The drive is always "KINGSTON"
  and the demo playlist is "Warehouse — Oct 2026". None of it is a
  compatibility claim.

## How to read the mockups

`design/*.dc.html` are self-contained HTML pages from a design canvas. Each
file is one frame, and `design/canvas.json` lists each frame's title, size and
position. They use a small template runtime (`{{holes}}`, `<sc-for>`,
`<sc-if>`, `<dc-import>`), and each file's logic is the `class Component`
script at its end. They won't render without that runtime, so read them as
source. The markup gives copy, spacing and structure. The script gives state
and behavior.

The visual language is the GPUI Kit default dark theme.
`design/ds/gpui-kit/tokens.json` maps the mockups' hex values to token names.
In GPUI, read every color, radius and spacing value from `cx.theme()` by
semantic role. Don't copy hex values into code.

| Mockup value | Token in `tokens.json` |
| --- | --- |
| `#0a0a0a` surface, `#fafafa` text | `background`, `foreground` |
| `#a3a3a3` secondary text | `muted-foreground` |
| `#262626` hairlines, secondary fills | `border`, `secondary-background` |
| `#2f2f2f` input outlines | `input-border` |
| `#171717` title and status bars | `title_bar-background`, `status_bar-background` |
| `#1e40af33` fill + `#1d4ed8` inset ring | `list-active-background`, `list-active-border` (selection) |
| `#fafafa` button with dark text | `primary-background` / `primary-foreground` (one default commit per surface) |
| `#f87171` / `#facc15` / `#4ade80` | `danger-background` / `warning-background` / `success-background`, always with an icon or word |
| `#1d4ed8`, `#3b82f6`, `#93c5fd` | `chart-4`, `chart-2`, `chart-1` (waveforms and other data only) |
| Hot-cue and tag colors | `base-red`, `base-yellow`, `base-green`, `base-cyan`, `base-blue`, `base-magenta` (user data, not decoration) |

## Screens

Each screen is a view or dialog in one desktop window. A dialog opens over
the screen that launched it and returns focus there when it closes. Dialogs
never stack. PRD §5 lists the required purpose of each surface.

| Frame | What it is | Opened from |
| --- | --- | --- |
| `Main.dc.html` | **Library.** Sidebar, playlist header, selected-track panel with waveform, track table, status bar | App start |
| `LibrarySplit.dc.html` | Library with **Split view** on: a source pane (collection or another playlist) to drag or add tracks from | Split view toggle |
| `EditMetadata.dc.html` | **Edit track info** dialog | Pencil on a row, or double-click |
| `CueEditor.dc.html` | **Cue editor**: overview and zoomed waveform, beat grid, hot-cue pads, memory cue list, inspector | Edit cues in the track panel |
| `Tags.dc.html` | **Tags**: categories, tag chips and "find tracks by tag" | Sidebar › Tags |
| `SmartPlaylist.dc.html` | **Smart playlist** rule editor with live preview | Sidebar smart playlist |
| `ImportReview.dc.html` | **Import music**: review, progress, results | Import › Music files… |
| `Migrate.dc.html` | **Import from rekordbox**: source, review, import, done | Import › rekordbox library… |
| `Analysis.dc.html` | **Analysis** queue and settings | Analyze, or Sidebar › Needs analysis |
| `Relink.dc.html` | **Missing files**: proposed matches, different-audio decisions, relinking | Sidebar › Missing files, Export's Locate…, Migrate's Relink… |
| `Settings.dc.html` | **Settings**: General, Library, Tags and files, Backup and restore | Gear in the toolbar |
| `Export.dc.html` | **Device page, Export tab**: storage, selections, settings, decisions summary | Sidebar › Devices › KINGSTON |
| `DeviceHistory.dc.html` | **Device page, History tab**: past exports, repair, drive backups | History tab |
| `HistoryInspect.dc.html` | History when a drive check finds unexpected changes | (state of History) |
| `HistoryUnproven.dc.html` | History when a file's ownership can't be proven | (state of History) |
| `HistoryRecovery.dc.html` | History when the drive needs recovery | (state of History) |
| `DeviceSetup.dc.html` | **Device setup**: player profiles, effective formats, My Tag categories, existing library | Export › Device setup… |
| `ExportReview.dc.html` | **Export review**: preflight that must resolve every open decision | Review and export… |
| `ExportResults.dc.html` | **Export results** and follow-up | View results…, History › Open report… |

### Suggested GPUI Kit components

| Need | Component |
| --- | --- |
| Window chrome, toolbar, status bar | `TitleBar`, `Toolbar`, `StatusBar` |
| Sidebar with sections and folders | `Sidebar` (with `Tree` for playlist folders) |
| Track tables, export review, history lists | `Table` (virtualized for the collection) |
| Split view, resizable panes | `Resizable` |
| Device page tabs, segmented controls | `TabBar` / tab segmented variant |
| Dialogs and confirmations | `Dialog` (alert variant for destructive confirmations) |
| Import menu, row menus | `DropdownMenu`, `ContextMenu` |
| Tag picker | `Popover` |
| Forms | `Input`, `Select`, `Checkbox`, `Radio`, `Switch`, `Stepper` for wizards |
| Progress, statuses | `Progress`, `Tag`, `Badge`, `Tooltip` |
| Settings window | `setting` components |
| Waveforms, beat grid, cue flags | Custom elements (paint), not HTML-like layout |

## How the mockups apply the PRD

These notes explain the mockups' behavior. They restate PRD requirements in
screen terms. They are not additional rules.

### Library and editing

- **Tag writing** follows the policy in Settings › Tags and files (`META-03`).
  Edit track info reports saving to Recrate separately from writing tags to
  the file. Cue points and beat grids are never written into audio files.
- **Unsaved edits.** With changes, closing turns the footer into "Discard N
  unsaved changes?" with **Keep editing** and **Discard**.
- **Split view** (`ORG-01`, `ORG-02`). Tracks can be added by dragging or with
  a + button on each row. A track already in the playlist shows "In
  playlist". Adding it again asks "Add it again?" in the status bar, so a
  duplicate occurrence is always an explicit choice. Every add can be undone.
  An occurrence only records membership and order. Tags, metadata and
  preparation belong to the track, so every occurrence shows the same tags.
  Tagging one occurrence tags the track, and undoing an add leaves nothing
  behind.
- **Smart playlists** (`ORG-03`, `ORG-04`). Players get an ordinary playlist
  of the matches, frozen into the reviewed export plan.

### Players, formats and cues

- **Device setup chooses player profiles, not "older" or "newer" players**
  (`DEV-02`, `DEV-03`). Each profile is labelled Tested, Untested or Not
  supported. Not supported can't be chosen, and choosing Untested shows a
  warning. The formats written are derived from the chosen profiles, and the
  same formats appear on Export, Review, Results and History.
- **Cue limits are per profile** (`PREP-04`, `PREP-05`). Recrate keeps every
  memory cue. The Cue editor's "Use" checkbox picks up to the profile's limit
  (10 in the sample data) for export. Plus isn't assumed to show more. A
  track over the limit with no choice blocks export until the user picks cues
  or accepts "first 10 by position", which names the bars it covers.
- **My Tag** exports only the categories the user chooses in Device setup, up
  to the profile's limit. The categories left out are named (`ORG-05`).
- **FLAC conversion** is offered only when a chosen profile can't play FLAC,
  and names that player (`EXP-06`).

### Export and recovery

- **One plan carries decisions between screens** (`EXP-02`–`EXP-04`).
  - Export builds the plan from the current selection and Device setup. The
    plan holds every operation for the selected playlists (copies, updates,
    missing files, cue decisions), the exact removal set with shared tracks
    counted once, the replacement scope and the formats.
  - Review draws its rows, counts, decisions and confirmations from that
    plan. Its confirmation names the playlists being removed.
  - Back and Close keep Review's decisions. Approvals to delete only carry
    over while the plan is the same. Any change to what's selected, removed
    or replaced makes a new plan and asks again.
  - Exporting turns the plan and Review's decisions into a run record, one
    row per operation. Export's summary, Results and History all read run
    records in that one shape.
  - Results hands its changes back to the record's owner (Export or
    History) when it closes: retried files, relinks and ejection. Reopening
    a report shows those changes, and History's stats and statuses follow
    them.
  - History opens the selected run's own report. Ejecting from any report
    ejects the drive for every screen, which closes the write gate.
  - Reopening Device setup shows the drive's saved setup.
- **Every open decision gates export** (`EXP-03`). Missing files, cue choices,
  removals and replacement are listed under "Needs your decision" on Export,
  and each is a checkbox or action in Export review. Export stays disabled
  until all are resolved. Missing tracks are left out only after "Export
  without these tracks" is ticked.
- **Keep versus replace** (`DEV-04`, `SAFE-09`). Keep preserves the existing
  library's semantics and refuses the write if that can't be done. Replace
  shows its exact scope: what's removed from the drive's library, what's
  kept on the drive (rekordbox's audio, by default), and what's backed up.
  Deleting that audio is a separate, unticked option. Replace, and audio
  deletion if chosen, each need their own approval in Export review. Review
  lists all 137 audio files, each with its own checkbox and none ticked. Only
  ticked files are deleted, after each is copied to the computer. With none
  ticked, all of them stay.
  Replacing never claims the computer has another copy.
- **Interrupted export** (`REC-03`, `REC-04`). History shows the last known
  state until a read-only check runs. Resume and Discard stay disabled until
  then.
- **Discard needs proof of ownership** (`SAFE-04`, `SAFE-08`). The check
  proves a file is this export's when its path was empty in the pre-export
  inventory and its content hash matches the hash Recrate recorded after
  writing it. For an incomplete file, the path was reserved before copying
  and the written part matches the source. Name, size and time appear only for
  reference. Proof and library references are checked again just before
  deleting. A file without proof moves History to "Recrate stopped here"
  (`HistoryUnproven`), and nothing is deleted.
- **Unexpected changes** stop Resume and Discard and offer Save report…,
  Show drive and Check again (`REC-05`).
- **Disconnected during a library write** means Recovery needed (`REC-06`).
  History never claims the old library is intact. It restores from the
  verified backup and reads it back before it reports recovery.
- **One write gate for the drive** (`SAFE-07`, Safety clarification 9). Every
  action that writes to the drive shares the same gate. That covers Resume,
  Discard, Restore… on any backup, Retry in a report, Eject, and a
  confirmation that's already open. The gate is closed when:
  - the drive isn't connected;
  - it needs recovery or inspection;
  - an interrupted export is still unresolved.

  Unplugging invalidates an earlier check, so Resume and Discard need a new
  check after reconnecting.

### Analysis and missing files

- **Changing analysis defaults never unlocks or queues tracks** (`AN-03`).
  Reanalyzing locked tracks is a separate, scoped action in the queue, and
  the replaced grid is kept so it can be restored.
- **A grid lock protects only the grid.** A locked track can still get
  key-only or waveform-only reanalysis. Row actions (Unlock…, Keep locked,
  Retry) never resume a paused queue; only Resume does.
- **Different audio** (`HEALTH-03`). A proposed match whose audio or length
  differs asks the user to choose one of three options:
  - relink, reanalyze the grid and mark cues "Check";
  - relink and keep everything as it is;
  - import the file as a separate track.

  Relink stays disabled until the user chooses. The results keep three
  outcomes apart: relinked, imported as a new track, and still missing.
  Importing the other file keeps the original missing and in its playlists.
  The summary states what happened to preparation for each choice. If the
  original is relinked later, the outcome says so. The action button names
  what it will do ("Relink 2 tracks", "Import 1 track"), and it stays
  disabled on the results screen too while a different-audio choice is
  open.
- **Removing a missing track** from the library deletes no file, on the
  computer or on any drive (`HEALTH-04`).

## Mockup shortcuts

Don't implement these as drawn. They are simplifications or open gaps in
the mockups.

- **Fixed canvas sizes.** Frames are 1440×900 or fixed dialog sizes. The
  product must work at smaller window sizes and with scaled text.
- **macOS only and dark only.** Paths, "Show drive in Finder" and shortcuts
  are macOS. Windows paths and shortcuts, and light or system appearance,
  aren't drawn.
- **Sample player profiles.** Model names, firmware, hot-cue counts, the
  10-cue limit and FLAC support are illustrative. Real values come from the
  qualification matrix.
- **Edit track info always shows both Save buttons.** It should follow the
  tag-writing policy.
- **Choose cues…** in Export review links to the Cue editor. Review keeps its
  decisions on Back and Close, but the round trip through the Cue editor and
  the recheck on return aren't drawn.
- **Plan freshness is by selection only.** A changed selection or setup
  invalidates approvals to delete. Source edits and drive reconnects, which
  should also invalidate them (`EXP-04`), aren't simulated.
- **Progress closes by Cancel.** Closing a job without cancelling it,
  reopening its progress, and recovery after restart aren't drawn.
- **Populated states only.** Most frames show working examples. Empty,
  error and offline states are listed under Not yet designed.

## Not yet designed

Taken from the PRD review. Most of these extend existing screens rather than
adding pages.

- **Library organization and browsing:** creating, renaming, moving,
  duplicating and deleting playlists and folders; removing an occurrence
  versus removing a collection record; reorder and insertion feedback;
  multi-selection; structured filters; configurable columns; showing sorted
  view versus stored order. (`LIB-05`, `BROWSE-01`, `ORG-01`, `ORG-02`)
- **Bulk metadata and external changes:** mixed-value editing with explicit
  field selection; the remaining metadata fields; removing artwork; "saved
  locally, file write failed or pending"; reviewing tag rereads and
  restoring original tags. (`META-01`–`META-05`)
- **Import and migration:** inspectable duplicate evidence and what
  replacement changes; scan and cancel states; partial cancellation results;
  separate reviews for desktop, XML and USB sources; which grid stays active
  with "keep both"; backup failure and retry. (`LIB-01`–`LIB-06`,
  `MIG-01`–`MIG-05`)
- **Preparation editor:** grid nudge and alignment; half and double tempo
  correction; lock and unlock; variable-tempo anchors and sections with a
  preview of the affected span; editable cue positions and loop bounds;
  active-loop intent; undo and dirty-exit confirmation; output volume. The
  rendering mode of the three-band waveform, what its bands mean, and its
  unavailable and loading states. (`PREP-01`–`PREP-03`)
- **Analysis queue:** cancel and cancelled states; partial results per
  component; reviewing stale or protected results; interrupted work;
  preparation provenance. (`AN-02`–`AN-04`)
- **Smart playlists and tags:** invalid-rule errors distinct from empty
  results; deeper nested groups; missing-value behavior; a proper tag picker;
  category settings; review of what a deletion affects; assignment controls;
  keyboard reordering. (`ORG-03`, `ORG-05`)
- **Export planning:** preservation conflicts that block the write; selecting
  individual tracks; stale plans and revalidation; conversion choices; peak
  space needed on the drive and the computer, including staging and backups;
  review and results for plain folders and M3U. (`DEV-02`–`DEV-04`,
  `EXP-01`–`EXP-07`)
- **Export progress and recovery:** separate statuses for legacy, Plus and
  shared resources; partial dual-format publication; stopping at a safe
  boundary; a missing or corrupt backup; a failed restore; repeated
  disconnection; the OS reporting the drive busy or failing to eject it.
  (`SAFE-05`–`SAFE-10`, `REC-01`–`REC-08`)
- **Local backup and relocation:** snapshot validation and compatibility;
  restore progress and failure; unavailable backup destinations; protected
  retention; a real library-move workflow. (`SET-02`–`SET-05`)
- **Computer handoff (new workflow):** publishing and adopting a snapshot;
  managed versus referenced audio scope; downloading audio on demand and
  Download all; incomplete delivery; divergent branches; unavailable
  removable catalogs. (Decision log D-07, D-08)
- **Cross-screen states:**
  - First launch and an empty library; no results across the collection; no
    connected drive.
  - An offline volume versus a missing file versus a permission failure.
  - Read-only or unsupported drives, and not enough space.
  - Keyboard and focus behavior, smaller windows, scaled text.
  - Windows paths and shortcuts, and light-theme examples.

## Interface language

- Sentence case everywhere. Buttons name the result in one or two words
  (**Export**, **Relink 3 tracks**, **Check drive**).
- Commands that open another dialog end with "…" (**Locate…**,
  **Device setup…**).
- Destructive confirmations name the object and use the result verb
  (**Discard copies…**, then **Discard copies**).
- Errors say what happened and how to recover. Don't write "successfully"
  or "Are you sure".
- Status color always comes with an icon or a word.
- Desktop conventions: default arrow cursor on buttons, one primary button per
  surface, everything keyboard-reachable, no hover-only actions.
- Release UI is English, with Unicode metadata and localization-ready text.
  Which other languages to ship is an open release decision (PRD §3.2).

## Kickoff prompt for an implementing agent

> Produce the technical specification and architecture for Recrate that
> `docs/recrate-prd.md` §14 asks for, not production code. The PRD,
> `docs/recrate-decisions.md` and the architectures in `docs/` are the
> authority. Don't treat open gates as decided. Use `ui/README.md` to read the
> mockups in `ui/design/*.dc.html` as source; they don't render on their own.
> Map each PRD requirement and each board to components, workflows and
> tests. Treat the README's Mockup shortcuts and Not yet designed lists as
> known gaps, not as behavior to copy. The stack is Rust, SQLite (rusqlite)
> and `gpui-kit`. Keep application code out of `gpui-base` and
> `gpui-component`.
