# Recrate design handoff

Recrate is a desktop DJ library manager: an alternative to rekordbox for
organizing playlists, setting cue points and exporting to USB drives for
Pioneer CDJ and XDJ players. It is not a performance app. There are no decks,
mixer or effects.

This folder is the design handoff for building Recrate with GPUI Kit. It holds
a written spec (this file) and the interactive mockups it was derived from
(`design/`). The mockups show layout, copy and behavior. This file states the
decisions behind them. When the two disagree, this file wins.

## Before you start

1. Read the repository's [Design Guides](../website/docs/design-guides.md) and
   [Coding Guides](../website/docs/coding-guides.md). They are requirements.
2. Recrate is an application built on `gpui-kit`. Put application code in its
   own crate or example. Do not add Recrate-specific behavior to `gpui-base`
   or `gpui-component`.
3. Treat the **Contracts** section as acceptance criteria. Most of them protect
   users' USB drives and libraries from data loss.
4. Everything in the mockups is sample data: track names, counts, sizes, dates
   and paths. The drive is always "KINGSTON" and the demo playlist is
   "Warehouse — Oct 2026".

## How to read the mockups

`design/*.dc.html` are self-contained HTML pages from a design canvas. Each
file is one frame. `design/canvas.json` lists every frame with its title,
size and position. They use a small template runtime (`{{holes}}`,
`<sc-for>`, `<sc-if>`, `<dc-import>`), and the logic lives in the
`class Component` script at the end of each file. They will not render
without that runtime, so read them as source. The markup gives exact copy,
spacing and structure. The script gives state and behavior.

The visual language is the GPUI Kit default dark theme. The hex values in
the mockups are that theme's tokens; `design/ds/gpui-kit/tokens.json` maps
them to names. In GPUI, read every color, radius and spacing value from
`cx.theme()` by semantic role. Do not copy hex values into code.

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

Every screen is a view or dialog in one desktop window. Dialogs open over the
screen that launched them, and closing returns there. They never stack: at
most one dialog is open at a time.

| Frame | What it is | Opened from |
| --- | --- | --- |
| `Main.dc.html` | **Library.** Sidebar, playlist header, selected-track panel with waveform, track table, status bar | App start |
| `LibrarySplit.dc.html` | Library with **Split view** on: a source pane (collection or another playlist) to drag or add tracks from | Split view toggle |
| `EditMetadata.dc.html` | **Edit track info** dialog | Pencil on a row, or double-click |
| `CueEditor.dc.html` | **Cue editor**: overview and zoomed waveform, beat grid, 8 hot-cue pads, memory cue list, inspector | Edit cues in the track panel |
| `Tags.dc.html` | **Tags**: categories, tag chips, and a "find tracks by tag" query | Sidebar › Tags |
| `SmartPlaylist.dc.html` | **Smart playlist** rule editor with live preview | Sidebar smart playlist |
| `ImportReview.dc.html` | **Import music**: review, progress, results | Import › Music files… |
| `Migrate.dc.html` | **Import from rekordbox**: source, review, import, done | Import › rekordbox library… |
| `Analysis.dc.html` | **Analysis** queue and settings | Analyze, or Sidebar › Needs analysis |
| `Relink.dc.html` | **Missing files**: proposed matches and relinking | Sidebar › Missing files, Export's Locate…, Migrate's Relink… |
| `Settings.dc.html` | **Settings**: General, Library, Tags and files, Backup and restore | Gear in the toolbar (⌘,) |
| `Export.dc.html` | **Device page, Export tab**: storage, playlists to export, settings | Sidebar › Devices › KINGSTON |
| `DeviceHistory.dc.html` | **Device page, History tab**: past exports, repair, drive backups | History tab |
| `HistoryInspect.dc.html` | History when a drive check finds unexpected changes | (state of History) |
| `HistoryRecovery.dc.html` | History when the drive needs recovery | (state of History) |
| `DeviceSetup.dc.html` | **Device setup**: players, library formats, existing rekordbox library | Export › Device setup… |
| `ExportReview.dc.html` | **Export review**: preflight before an export that removes anything | Review and export… / Review changes… |
| `ExportResults.dc.html` | **Export results** and recovery | View results…, History › Open report… |

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

## Core concepts

- **Collection, playlists, folders.** Playlists live in folders. Smart
  playlists are rule-based and marked "Smart" in the sidebar.
- **Hot cues and memory cues are different things.**
  - Hot cues are triggers: 8 pads (A–H) per track, each with color, name and
    optional loop. On the waveform they are flags along the top.
  - Memory cues are markers with no pads and no limit in Recrate. On the
    waveform they are triangles along the bottom.
  - Copying one kind to the other never moves or removes the original.
- **Tags** belong to user-defined, colored, reorderable categories. When
  finding tracks, tags in the same category combine with OR, and different
  categories combine with AND.
- **Device libraries.**
  - A USB drive can hold Device Library (older players), Device Library Plus
    (newer players), or both.
  - Which ones a drive gets is a property of the drive, set in Device setup
    ("Older players", "Newer players", "Both / not sure"). Export only shows
    what will be written.
  - Plain folders (audio files plus M3U, no cues) is a separate export mode.

## Contracts

These are the rules the design depends on. Implement them as behavior, and
test them where the Coding Guides call for tests.

### Library and editing

1. **Tag writing follows one policy** (Settings › Tags and files): only change
   Recrate's library, choose each time, or always write to files too.
   - With "choose each time", Edit track info offers **Save** (library only)
     and **Save and write tags** (also writes ID3 tags or Vorbis comments).
   - Cue points and beat grids are never written into audio files.
   - Known gap: the Edit track info mockup always shows both buttons. Make it
     follow the policy.
2. **Unsaved edits.** Cancel, the close button and Escape close at once when
   nothing changed. With changes, the footer turns into "Discard N unsaved
   changes?" with **Keep editing** and **Discard**. It is not a second dialog.
3. **Split view** hides Tags, Hot cues and Added in the playlist table. Adding
   works both by dragging and by a + button on each row, so it never depends
   on dragging. Tracks already in the playlist can't be added twice.
   Every add can be undone from the status bar.
4. **Smart playlists.**
   - Each rule row shows how many tracks are left after it (or how many it
     matches, for "any").
   - Unfinished rules are ignored rather than emptying the result.
   - Players can't evaluate rules. On export, a smart playlist is written as
     an ordinary playlist of its current matches, either refreshed on every
     export or frozen at the first one (the user chooses).

### Cues and device limits

5. **Device Library holds at most 10 memory cues per track.**
   - Recrate keeps every memory cue. In the Cue editor, a "DL" checkbox on
     each memory cue chooses which ones go to Device Library, with at most 10
     ticked.
   - Device Library Plus gets every memory cue. Don't claim players will show
     them all.
   - Export must not silently keep "the first 10". If a track over the limit
     has no choice, Export review blocks the export until the user chooses
     cues or ticks "Use <track>'s first 10 for Device Library".

### Export safety

6. **Review before deleting from a drive.** If an export would remove
   anything from the drive, Export goes through Export review. Export stays
   disabled until the user ticks the removal confirmation, and until every
   memory cue choice from contract 5 is resolved.
7. **Shared drive library.** A drive has one shared library, so adding
   Recrate's playlists means rewriting it even when the user keeps
   rekordbox's.
   - Before every export, back up each library and analysis file that will
     change.
   - After writing, read the library back and compare it. If anything of
     rekordbox's is missing or different, mark the export failed and restore
     the backup.
   - Only write library versions Recrate recognizes. Refuse unfamiliar
     formats.
   - The "Recrate's playlist" marker is a convenience for listing and
     removal. It is not the safety mechanism.
8. **Removing a playlist** frees only tracks no other playlist on the drive
   uses. Space estimates must reflect that.
9. **Interrupted export: check before anything else.**
   - History shows an interrupted export as its last known state ("last known
     at 22:10"), not as facts about the drive.
   - Check drive comes first and requires the drive to be connected. Resume
     and Discard stay disabled until a check has run.
10. **Discard deletes only what this export provably created.** Delete only
    files the check identified as created by this export (by name, size and
    time) and not used by either device library. Files that were already on
    the drive, or that any library points to, are kept.
11. **An unexpected check result stops everything.** For example, the
    library changed after the disconnect, or the export's files are now used
    by another library.
    - Show "Recrate stopped here" with the differences.
    - Offer only Save report…, Show drive in Finder and Check again.
    - Resume and Discard are not offered.
12. **Disconnected during a library write means recovery is needed.**
    - Show "Recovery needed". Don't claim the old library is intact or that a
      rollback happened.
    - Step 1: reconnect the drive. Step 2: Check and restore (compare with the
      pre-export backup, put back files that don't match, read them back).
    - Tell the user not to use the drive in a player until recovery is done.
13. **Every export's backup is listed in History** with Restore…. Restoring
    first backs up the current drive library.

### Library-wide safety

14. **Recrate library backups** (Settings › Backup and restore) cover
    playlists, cues, beat grids, tags and analysis, but not audio files.
    - Back up automatically on a schedule, and before large imports,
      reanalysis and restores.
    - A restore's confirmation says what will be lost, and backs up the
      current library first.
15. **Missing files are a collection problem.** They are reachable from the
    Library sidebar (Missing files, with a count), Export's Locate… and the
    migration's Relink….
    - Weak matches start unticked.
    - Relink applies only to ticked matches.
16. **rekordbox migration** reads a copy and never changes rekordbox or its
    files.
    - It backs up Recrate's own library first.
    - It shows a mapping table (imported / imported with changes / not
      imported) before anything is imported.
    - It asks how to resolve tracks that exist in both apps.
    - It can lock imported beat grids.
    - Its Done step links straight to Missing files.
17. **Analysis never replaces locked or user-edited beat grids** unless the
    user unlocks them. Reanalyze warns when unlocked grid edits will be
    replaced.

## Interface language

- Sentence case everywhere. Buttons name the result in one or two words
  (**Export**, **Relink 3 tracks**, **Check drive**).
- Commands that open another dialog end with "…" (**Locate…**,
  **Device setup…**).
- Destructive confirmations name the object and use the result verb
  (**Discard copies…**, then **Discard copies**).
- Errors say what happened and how to recover. No "successfully", no
  "Are you sure".
- Status color always comes with an icon or a word.
- Desktop conventions: default arrow cursor on buttons, one primary button per
  surface, keyboard reachable everything, no hover-only actions.
- Localization: `en`, `zh-CN`, `zh-HK` (see the root `CLAUDE.md`).

## Open items

- **Supported players list.** Device setup shows placeholders
  (`[Player model]`, `[x.yz]`, `Recrate [version]`). Fill it from real,
  versioned compatibility testing before release. Until then, don't claim
  compatibility beyond what's tested.
- **rekordbox details to verify** against the rekordbox version you target:
  - The XML export menu path.
  - What XML omits (for example My Tag).
  - That phrase analysis, history and sampler are not imported.
  - The mapping of intelligent playlist conditions.
- **Device library facts to verify:** hot cues per track (the design assumes
  8) and track colors on each format.
- **Not yet designed:**
  - Import history (reopening past import reports).
  - Batch editing and tag "recipes" for many tracks at once.
  - A proper tag picker in smart playlist rules (they are typed text today).
  - Playlist reordering by drag.
  - The category settings dialog on the Tags page.
- **Sample data** throughout: replace with real data sources.

## Kickoff prompt for an implementing agent

> Build Recrate, a desktop DJ library manager, as an application on
> `gpui-kit` in this repository. Read `recrate/README.md` first. It is the
> spec, and its Contracts section is the acceptance criteria. Then read the
> repository's Design Guides and Coding Guides and follow them. Use
> `recrate/design/*.dc.html` as the reference for layout, copy and interaction
> (read them as source; they don't render on their own). Start with the
> Library screen (`Main.dc.html`) using sample data. Then add the Cue editor,
> the device page (Export and History) with its dialogs, and the remaining
> dialogs. Use GPUI Kit components and theme tokens rather than the mockups'
> hex values. Don't modify `gpui-base` or `gpui-component` for app-specific
> needs. Ask before deciding anything the Open items section leaves open.
