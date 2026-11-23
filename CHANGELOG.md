# Changelog

Release notes for the KanuniLabs components. Also published at
<https://www.kanunilabs.com/changelog>.

> This file is generated from the source of truth in the product repository.
> Please do not edit it by hand — open an issue instead if something is wrong.

PivotGrid and DataGrid are versioned independently, so entries name the
component they belong to. Entries before August 2026 predate DataGrid and are
PivotGrid releases.

## PivotGrid v1.11.16 — November 23, 2026

`@kanunilabs/pivotgrid-core@1.11.16` · `@kanunilabs/pivotgrid@1.6.16` · `@kanunilabs/pivotgrid-react@1.8.16` · `@kanunilabs/pivotgrid-enterprise@1.7.16` · `@kanunilabs/pivotgrid-react-enterprise@1.8.16`

_Very large pivots: the node menu's "Collapse all" no longer needs more memory than the open tree it closes, a large collapsed state is saved as one compact entry, and feeding rows into a fully expanded report no longer holds two whole trees at once._

### Before you upgrade

- `maxLiveOpenNodes` is on by default (500,000): while rows stream into a fully expanded report (`'*'`) whose tree is larger than that, live computes are held back; the screen keeps its last result, `isComputing` stays true, and the full result is computed when the feed ends (`flushAsyncTransactions()`, `setLoading(false)`), on any change by the reader or your code, for an export, or after about 2 seconds without new rows. Pass `maxLiveOpenNodes={0}` (or `Infinity`) for the behaviour of 1.11.15. Smaller reports are not affected.
- React grids and Enterprise grids: with more than 5,000 row groups closed under "Expand all", the saved state (`getPersistedState`, `stateStoring`, saved views) holds the closed rows as one text entry inside `expandedPaths` instead of one entry per group. The entry is plain text and survives `JSON.stringify`; treat it as opaque and pass `expandedPaths` back as you received it. Code that reads or filters the strings in `expandedPaths` should read `isExpanded` on the result's nodes instead. With fewer closed groups the state is saved exactly as before.
- A state saved in that compact form and opened by 1.11.15 or older opens with its rows collapsed (nothing throws). Update every install together if reports are shared.
- `getState().expandedPaths` is the live set, not a saved state: while the closed rows are kept compactly it holds `'*'` and one handle. A copy handed back to `setExpandedPaths` on the same page restores the rows; stored and read in another session it opens every row and reports through `onError`. Use `getPersistedState()` or `stateStoring` to keep the open rows across sessions.

### Added

- `maxLiveOpenNodes` (on by default, 500,000; `0` or `Infinity` turns it off): the most row nodes a live-update compute may build while rows stream into a fully expanded report. See the Large data guide.
- `layout.rowFieldChips: 'aligned' | 'packed'` (default `'aligned'`, today's look): `'packed'` puts the row-field chips side by side with a small gap instead of one chip per column width. Columns keep their widths and their resize grips. React and plain JS; not saved with the state.
- The theming guide shows how to paint the empty band under the row headers with `--pivot-row-area-empty-bg` (React).

### Changed

- React and Enterprise grids: the node menu's "Collapse all" on a large expanded tree keeps the closed rows as one bit per group instead of one text entry per group, sends the worker one command, and gets back only the rows that are still visible. Measured in Node on 300,000 rows with a per-row field in the middle of the row fields (906,000 groups): the main-thread heap peaks at 945 MB instead of about 1,900 MB and settles at 238 MB instead of about 1,700 MB, in 1.5 s. The meaning is unchanged: groups that existed are closed, and a member or a level that arrives later comes open.
- The same closed rows are saved as one compressed text entry: on that report about 3.3 MB of JSON instead of 54 MB, restored in about 1 s instead of 7 s, with a peak of 192 MB instead of 1,091 MB. A save right after opening or closing one branch adds the change to the entry instead of compressing the tree again.
- Moving, adding or removing a field, a filter, a data refresh and undo carry the closed rows over without turning them back into text. Closed groups the current data does not hold are remembered and come back closed.
- The compact form is not used (the grid works as in 1.11.15) while a search is active, with a row top-N, value or label filter, a colour scale or other range-based conditional format, a running total or difference display, a `serverSource`, or member texts containing `|`; nor in the free plain-JavaScript grid and the Angular wrapper. When the search or the setting is removed the grid returns to the compact form.

### Fixed

- A grid with `stateStoring` disabled no longer serialises its whole state on every change.

## PivotGrid v1.11.15 — November 21, 2026

`@kanunilabs/pivotgrid-core@1.11.15` · `@kanunilabs/pivotgrid@1.6.15` · `@kanunilabs/pivotgrid-react@1.8.15` · `@kanunilabs/pivotgrid-enterprise@1.7.15` · `@kanunilabs/pivotgrid-react-enterprise@1.8.15`

_Very large pivots: every row of a pivot taller than the browser's layout limit is reachable, loading by feed is no longer quadratic, and the worker needs about half the memory to compute an opened pivot._

### Before you upgrade

- Nothing to change for a grid that worked on 1.11.14. Expanding, collapsing and the saved state behave and are stored exactly as before.
- Above 12,000,000 px of rows (about 570,000 rows at 21 px) the body keeps that height and the scroll position maps to the rows, so one pixel of the scrollbar is more than one row there; the wheel, the keyboard and the saved scroll position move by rows as before. Below that height nothing changes.
- Free plain-JavaScript grid only: four Enterprise button captions (Filter Builder, Toggle Chart View, Calculated Measure Designer, Edit Calculation) left the shared dictionary of the script-tag bundle. Enterprise grids show them in every language as before; a free grid whose app fills those buttons itself shows them in English unless its `dictionary` names them.

### Added

- `maxRestoredOpenNodes` and `onRestoredExpansionCollapsed` (off by default): an app can set a limit on the row nodes a restored open state may make the grid hold. Over it, the report opens collapsed, the saved open state is emptied and the app is told. What the reader opens is never limited. See the Large data guide.

### Changed

- A feed that only adds rows (`applyTransactionAsync({ add })`) no longer walks the whole row tree after every batch. The cost grew with the square of the row count: with a per-row field in the middle of the row fields, 2 million rows took 250 s to load instead of 75 s. In the engine alone, adding 400,000 rows in batches of 2,000 fell from 11.0 s to 0.9 s.
- Computing the data cells in the worker no longer builds a JavaScript Map or doubling arrays: on 1 million rows with every level open it is about three times faster and needs about half the peak memory (440 MB to 213 MB; with two column fields 1,420 MB to 523 MB). Results are bit-identical.
- Building the pivot in the worker finds each row's group through a typed table instead of a JavaScript Map: about a quarter faster on 1 million distinct rows, with less memory.

### Fixed

- A pivot whose rows are taller than the browser can lay out (about 1.6 million rows at 21 px in Chrome) can now be scrolled to its last row. The keyboard, the wheel and the saved scroll position follow the same mapping, and screen readers still get the real row numbers.
- `expandAll({ allLevels: true })` sets `isComputing` when it is called. It used to stay false while the worker counted the groups, so code that waited for the calculation right after the call saw an idle grid.

## DataGrid v1.10.20 — November 21, 2026

`@kanunilabs/datagrid-core@1.10.20` · `@kanunilabs/datagrid@1.9.20` · `@kanunilabs/datagrid-react@1.8.20` · `@kanunilabs/datagrid-enterprise@1.10.20` · `@kanunilabs/datagrid-react-enterprise@1.9.20`

_The header filter and the cell menu no longer close by themselves right after opening when the click had to scroll the grid first._

### Before you upgrade

- Nothing to change. One difference: a context menu taller than the screen no longer closes when the wheel scrolls the menu itself.

### Fixed

- React: the header filter, the custom filter and the cell menu no longer close by themselves when a click scrolls the grid to reveal the column or cell just before they open. Chrome reports that scroll one frame later, after the popup is up, and the popup took it for a scroll of the page. A scroll reported before the first frame after opening is now ignored; any later scroll outside the popup closes it as before.
- Popups, menus and dialogs take focus without scrolling the page or the grid (React and plain JavaScript).

## PivotGrid v1.11.14 — November 21, 2026

`@kanunilabs/pivotgrid-core@1.11.14` · `@kanunilabs/pivotgrid@1.6.14` · `@kanunilabs/pivotgrid-react@1.8.14` · `@kanunilabs/pivotgrid-enterprise@1.7.14` · `@kanunilabs/pivotgrid-react-enterprise@1.8.14`

_Large data in the browser: a new Enterprise `columnar` option keeps the rows as typed columns instead of row objects, so 1–2 million rows fit, and "Collapse all", the saved state and the calculation worker use far less memory on a pivot with a per-row field among the rows._

### Before you upgrade

- Without `columnar` (Enterprise) the grid keeps your row objects and behaves as in 1.11.13, with one difference: a `{ columns, rows }` chunk given to a grid that is not columnar now throws instead of silently adding nothing.
- With `columnar`, rows arrive as `{ columns, rows }` chunks (or as row objects, which are encoded and not kept) through `data`, `setData` and `applyTransaction(Async)({ add })`; `update` and `remove` throw in this mode for now, and `columnar` cannot be changed after the grid is built. A row-level `calculateExpression` receives a reused, read-only row: read values from it, do not keep it or write to it. `getRecordsForSelection()` and `cellClick` records still work: up to 50,000 rows they are an array built at once, above that a read-only list whose rows are built when read; for a large selection use a row source (`getSelectionRowSource()`).
- The data rule, in both modes: a new `data` value (an array, an empty array, or a chunk with new `rows` / `columns` arrays) replaces every row and drops adds still queued by `applyTransactionAsync`; the same value again changes nothing. Memoize a chunk you pass as a prop: a fresh `[]` or chunk on every render clears the streamed rows on every render (the grid warns once).
- `stateStoring` / `onSave` is now called only when a saved part changes (fields, layout, column widths, open groups), not on every result or progress tick. Code that used `onSave` as a "something happened" signal must listen to the grid's events instead.
- After "Collapse all" from a header menu with everything expanded, a last-level row's menu says "Collapse" (the click changes nothing, as before), and a field appended at the end of the rows afterwards shows that new level open. States saved by 1.11.13 that hold an entry per leaf keep those entries until the next "Expand all".
- The exported `FastTrie`'s `childrenMap` is now built when read and is read-only.

### Added

- Enterprise: `columnar` keeps the rows in a column store on the main thread (about 230–400 bytes per row instead of the size of your row objects). Selection details, custom summaries, row-level calculated fields, filters, exports and hidden React tabs work as in row mode; every feature is tested against row mode on the same data. See the new Large data guide.
- Enterprise: row sources read the rows behind a selection or a cell page by page, with or without `columnar`: `getSelectionRowSource()` on the React ref, `selectionRowSource`, `cellRowSource`, `rowStoreOf`. `toRowArray()` gives a DataGrid one small row handle per row, so sorting, the header filter, Excel export, column moves, range totals and copy work as over plain rows. The drill-down windows accept a row source and build only the page on screen.

### Changed

- "Collapse all" from the row or column header menu no longer writes a closed-branch entry for every leaf when everything was expanded: at 250k rows with a per-row field the step took 3.5 s and peaked 1.35 GB above where it started; it now takes 1.5 s and about 120 MB. What is shown, re-opening a branch and the saved state format are unchanged.
- The calculation worker keeps no text path and no map per group: with a per-row field (a barcode) among the row fields its memory fell from 269 MB to 14 MB at 250k rows, and rearranging the row fields takes half the time. Results are bit-identical.
- The open-group comparison no longer builds and keeps a sorted copy of every open path.

### Fixed

- The grid state was saved on every state change, also when nothing saved had changed: one collapse called `onSave` five times with the same state.
- React: a recompute kept the previous result alive through the column resize handles.
- React Enterprise: leaving out `data` handed the grid a new empty array on every render.
- After a rebuild that failed, the next compute could answer with empty cells instead of the error.
- New `data` given while a large first build was still running, followed by `applyTransactionAsync` adds, left the grid empty: the adds waited for a build of the replaced rows. A refresh that clears the data and streams the new rows right away could hit this.

## DataGrid v1.10.19 — November 21, 2026

`@kanunilabs/datagrid-core@1.10.19` · `@kanunilabs/datagrid@1.9.19` · `@kanunilabs/datagrid-react@1.8.19` · `@kanunilabs/datagrid-enterprise@1.10.19` · `@kanunilabs/datagrid-react-enterprise@1.9.19`

_`rowSourceDataSource` serves the lazy row model from an indexed row source, such as the rows behind a PivotGrid selection._

### Before you upgrade

- Nothing changes for grids over plain rows. Column inference (a grid without `columns`) now also reads inherited enumerable getters when a row has no own keys, so rows that are handle objects get their columns.

### Added

- `rowSourceDataSource(source)` serves the lazy (remote) row model from an indexed row source (`rowCount` and rows by range), with the client model's own filtering, search and sort. With a remote model a range selection covers only the rows already loaded; to show every row of a PivotGrid selection with complete range totals and copy, pass the row source's `toRowArray()` as `data` instead.

## PivotGrid v1.11.13 — November 19, 2026

`@kanunilabs/pivotgrid-core@1.11.13` · `@kanunilabs/pivotgrid@1.6.13` · `@kanunilabs/pivotgrid-react@1.8.13` · `@kanunilabs/pivotgrid-enterprise@1.7.13` · `@kanunilabs/pivotgrid-react-enterprise@1.8.13`

_A pivot shown again after React hid it no longer recomputes when nothing changed, and `layout.rowHeaderToggle: 'button'` lets a click on a row header select it without opening or closing the group._

### Before you upgrade

- A grid shown again after `<Activity mode="hidden">` with nothing changed fires no `engineStarted` / `engineCompleted` and keeps `isComputing` false; code that waited for a compute on every show must not. After a hide longer than the worker's 5 s keep-alive, the engine starts again with the first change that needs it, not on show.
- The built-in language objects (`en`, `tr`, ...) no longer carry `systemError`, `noConditionsAdded`, `exportGeneratedOn`, `saveChanges` and `loadingData` (read them with `pivotExtraText(locale, key)` from `@kanunilabs/pivotgrid-core`) nor the unused `type_*` texts. The grids show the same text, and a `dictionary` you pass still overrides them.
- `layout.rowHeaderToggle` defaults to `'cell'`: nothing changes unless you set `'button'`. It is an app option and is not saved with the grid state.

### Added

- `layout.rowHeaderToggle: 'cell' | 'button'`. With `'button'`, a click or tap on a row-header cell or on a collapsed group's label only selects it, and only the +/− box opens and closes the group, so a header can be selected for Selection Details or Ctrl+C without collapsing it. Space still toggles from the keyboard. On touch screens the box has a 44 px wide hit area, as tall as the row. React and plain JS.
- `pivotExtraText(locale, key)` in `@kanunilabs/pivotgrid-core`.

### Changed

- A grid shown again after React hid it (`<Activity mode="hidden">`) no longer recomputes when nothing changed while it was hidden: the last result is shown at once and no request reaches the worker. Shown within 5 s, the worker is kept; toggles under expand-all, selection, `getCell` and exporting the current view work without it.
- When something did change while the grid was hidden, the result on screen stays up and is replaced when the new one is ready, as for live updates: no loading overlay on show, and `isComputing` is true until then.

### Fixed

- A filter changed while the grid was still building (for example a saved state that an asynchronous `stateStoring.onLoad` returned late) was not applied: the grid could show unfiltered numbers under a correct filter chip.
- On touch screens a row-header cell no longer lets a long `cellTemplate` or the tree layout's connector line draw out of its cell.

## PivotGrid v1.11.12 — November 19, 2026

`@kanunilabs/pivotgrid-core@1.11.12` · `@kanunilabs/pivotgrid@1.6.12` · `@kanunilabs/pivotgrid-react@1.8.12` · `@kanunilabs/pivotgrid-enterprise@1.7.12` · `@kanunilabs/pivotgrid-react-enterprise@1.8.12`

_A React PivotGrid hidden by `<Activity mode="hidden">` (a kept-alive tab) comes back with its rows, layout and result instead of staying on "Loading" or showing the totals of the first rows only._

### Before you upgrade

- React: a grid inside `<Activity mode="hidden">` is suspended, not disposed, and recomputes when shown. React sets `ref.current` to `null` while a grid is hidden, so a feed that calls `ref.current?.applyTransactionAsync(...)` drops rows while the tab is hidden: keep the controller from the new `onReady(controller)` prop or `getController()` instead.
- React: an unmounted grid is still disposed. A grid unmounted while hidden is disposed after two missed 5 s checks; a grid whose hidden element a tab host removed from the page is rebuilt with its layout when shown (a reset, not a dead grid).
- Each live grid now has its own engine worker, also two grids with the same or no `gridId`. Saved reports, the stored layout and popup sizes are still kept per `gridId`, so give each kept-alive tab's pivot its own stable `gridId`.
- A request still waiting on a released engine worker now rejects with an `AbortError` instead of staying pending; an export started while a grid is suspended rejects the same way.

### Added

- `suspend()`, `resume()`, `isSuspended()`, `isDisposed()` and `getEngineKey()` on `PivotController`, and an `onReady(controller)` prop on the React grids, called once per controller.

### Fixed

- React: a pivot inside `<Activity mode="hidden">` was disposed by its effect cleanup and never rebuilt, so after the tab came back it stayed on "Loading", or reported done with the result of the first rows only. It now keeps its rows, layout, open branches, last result, queued transactions and listeners while hidden, and while it has changes it has not computed `isComputing` stays `true`: a hidden grid never reports a partial result as done.
- React: an open field chooser, menu or window of a grid hidden by `<Activity>` is hidden with it.
- Calling a disposed controller logs a warning once instead of doing nothing silently.
- Two live grids with the same or no `gridId` could share one engine worker and show each other's numbers.

## DataGrid v1.10.18 — November 19, 2026

`@kanunilabs/datagrid-core@1.10.18` · `@kanunilabs/datagrid@1.9.18` · `@kanunilabs/datagrid-react@1.8.18` · `@kanunilabs/datagrid-enterprise@1.10.18` · `@kanunilabs/datagrid-react-enterprise@1.9.18`

_A React DataGrid hidden by `<Activity mode="hidden">` keeps its sort, filters, grouping, selection, scroll position and unsaved edits; and the context menu no longer offers a Paste that cannot work on a plain-HTTP page._

### Before you upgrade

- React: `onReady` is called once per controller: not again on a re-render, on StrictMode's re-mount or when a hidden grid is shown, so listeners added there are added once. Code that relied on an inline `onReady` running after every render must move that work into an effect.
- React: `useDataGrid(options, { attachedTo })` takes the grid's root element as a second argument; without it the hook keeps its old behaviour and a hidden grid is rebuilt when shown. A grid whose hidden element a tab host removes from the page is still reset, and warns once in the console if it held unsaved edits.
- Enterprise: `styledExport` now merges its two keys into the Excel export defaults instead of replacing them, so defaults set in `onReady` (`stream`, `fileName`, `sheetName`) are kept.
- Enterprise: on a plain-HTTP page (no secure context, or no `navigator.clipboard.readText`) the context menu no longer shows Paste; Ctrl/Cmd+V works as before. A Paste item you list yourself in `rowMenu` is not hidden.

### Added

- `useHideWithGrid(ref)` hides an app's own portalled layer together with the grid, and `controller.getExportDefaults(format)` reads the current export defaults.

### Fixed

- React: a grid inside `<Activity mode="hidden">` was disposed and rebuilt when shown, losing sort, filters, grouping, selection, scroll position, rows changed through the API and unsaved batch or row edits. It now keeps its controller; a real unmount disposes it as before (`gridPreDestroyed`, a viewport source's `destroy()`), and a grid unmounted while hidden is disposed after two missed 5 s checks.
- React: hiding a grid neither saves nor discards an open editor; the editor gets its focus back when the grid is shown, and in row mode a row is not sent because the tab was switched.
- React: popups rendered through a portal (filter, menus, calendar, lookup lists, dialogs, a chart in the app's container) stay on screen no longer when the grid is hidden.
- Enterprise: the context menu's Paste read the async clipboard, which a plain-HTTP page does not have, so it could only fail; it is no longer offered there.

## DataGrid v1.10.17 — November 17, 2026

`@kanunilabs/datagrid-core@1.10.17` · `@kanunilabs/datagrid@1.9.17` · `@kanunilabs/datagrid-react@1.8.17` · `@kanunilabs/datagrid-enterprise@1.10.17` · `@kanunilabs/datagrid-react-enterprise@1.9.17`

_Long lists without a blank body on a jump to the end: draw every row (`virtualization.mode: false`, or `'auto'` under a threshold), or show row-height placeholder lines while the rows catch up (`virtualization.placeholderRows`); and copy, cut and paste that work on a plain-HTTP page, with `onCopy` / `onCopyError` (PivotGrid's names) to tell the app what became of a copy._

### Before you upgrade

- Nothing changes unless you set `virtualization.mode` or `virtualization.placeholderRows`: with neither, the grid draws the same rows, the same DOM and the same styles as in 1.10.16.
- A `virtualization.overscanY` that is not a finite number of 0 or more (`-1`, `NaN`) now means the default 5; before, a negative one meant 0. The same holds for the new `threshold` (default 2000).
- Every copy (Ctrl/Cmd+C, Ctrl/Cmd+X, the menu's Copy items, `writeClipboardText`) is written through the browser's copy event or `execCommand('copy')` first and `navigator.clipboard` only last, and it writes an HTML table (`text/html`) next to the tab-separated text, so a paste into Excel or Word keeps the cells; a paste into a plain text field gets the same text as before. A copy that is not a key's own reaches the clipboard one task later than before. Ctrl/Cmd+C still copies only while the keyboard focus is in the grid and not in a text field, as before. With `clipboard.sendToClipboard` nothing changes: the app gets the text and the system clipboard is left alone.
- `controller.writeClipboard(text)` answers `true` whenever there is text to write, also where no clipboard turns out to exist (it answered `false` on plain HTTP); the outcome arrives through `onCopy` / `onCopyError`. A copy with nothing to copy now calls `onCopyError` with `'empty-selection'`.
- Plain JS Enterprise: Ctrl/Cmd+V on a plain-HTTP page no longer prevents the key; for that one key the focus moves to an off-screen text area inside the grid and comes back, as in the React grid. A `focusin` / `focusout` listener of the app inside the grid sees that move. On HTTPS and localhost nothing changes.
- Ctrl/Cmd+C with no row selected or focused while text is selected on the page (with the mouse, in the grid or outside it) no longer calls `onCopyError` with `'empty-selection'`: the browser copies the text and the grid keeps out of it, as PivotGrid 1.11.10 does. `hasPageTextSelection()` is exported from `@kanunilabs/datagrid-core`.
- A page-level `copy` listener of the app now sees the grid's copies: Ctrl/Cmd+C and the menu's Copy items go through a real (or, for the menu, an `execCommand`-made) `copy` event, which the grid answers in the capture phase; a listener of the app that calls `clipboardData.setData` in the bubble phase writes over the grid's data and wins. Because a copy also writes an HTML table (`text/html`), a single copied cell pastes into Word or Outlook as a 1×1 table, not as plain text (the same in PivotGrid 1.11.10); a plain-text paste (Ctrl+Shift+V) gives the text.

### Fixed

- Ctrl/Cmd+C, the menu's Copy items and Ctrl/Cmd+X copied nothing on a plain-HTTP page (`http://10.0.x.x`): `navigator.clipboard` exists only on HTTPS and localhost, and the failure was swallowed. The copy now goes into the browser's copy event that follows the key, which every page has, and a copy that comes with no key event (the menu, `copySelection()`, a cut) through `execCommand('copy')`; `navigator.clipboard` is the last resort. A copy that still fails calls `onCopyError` (React, plain JS and both Enterprise wrappers).
- Ctrl/Cmd+V of the plain JS Enterprise grid pasted nothing on a plain-HTTP page: it read `navigator.clipboard`, which is not there, and swallowed the failure. It now takes the text from the browser's own paste event, as the React grid already did. The menu's Paste item still reads `navigator.clipboard`: a browser hands the clipboard to a page only on the user's own Ctrl/Cmd+V, so on plain HTTP that item cannot paste; the key does.

### Added

- `virtualization.mode` (`true` | `false` | `'auto'`, default `true`): `true` virtualizes the rows as before; `false` draws every row at once, so a jump to the end or a fast fling never shows an empty body; `'auto'` draws every row while the view has at most `virtualization.threshold` rows (default 2000) and virtualizes above that. The count is the rows the grid would draw, after filtering, with group headers and open groups. One DOM row per row: keep `false` to a few thousand rows. Ignored for a server source (`DataSource`, `viewportDatasource`), where drawing every row would fetch every row, and for a list too tall for the browser's canvas (past roughly a million rows). Rows only: columns stay virtualized (`overscanX`), so while every row is drawn each sideways scroll step redraws every row. Read when the grid is created: changing `virtualization` on a live grid has no effect. React, plain JS and both Enterprise wrappers.
- `virtualization.placeholderRows` (default `false`): while the rows are virtualized, the part of the body no drawn row covers yet shows an empty line at every row height instead of a blank surface, until the rows arrive. A background on the rows' spacer: no extra DOM, no measuring, aligned under pinned columns. Colours `--dg-placeholder-row-bg` (default `--dg-surface`) and `--dg-placeholder-row-line` (default `--dg-border`), so every theme and colour scheme matches. The lines follow the standard `rowHeight`; card view keeps the plain surface. The stylesheet of this release is needed.
- `controller.isRowVirtualized()`, `controller.showsPlaceholderRows()` and `DEFAULT_VIRTUALIZATION_THRESHOLD` (`@kanunilabs/datagrid-core`).
- `onCopy(info)` and `onCopyError(error, { reason })` (React props, plain JS config; both Enterprise wrappers pass them on), the same names and shapes as PivotGrid's. `onCopy` gets `{ text, html, rows, cols }` once a copy (Ctrl/Cmd+C, a menu Copy, a cut) reached the clipboard; it is not called when `clipboard.sendToClipboard` takes the text. `onCopyError` gets a `GridCopyError` and its `reason`: `'insecure-context'` (plain HTTP and no way to write), `'permission-denied'` (the browser refused), `'empty-selection'` (nothing to copy) or `'unsupported'`. The grid shows nothing itself: tell the user from here. `controller.setCopyCallbacks()`, `copyInfoFromTsv`, `writeCopyEvent`, `writeClipboardFallback`, `writeClipboardSoon` and `gridCopyError` are exported from `@kanunilabs/datagrid-core`, with the `GridCopyInfo`, `GridCopyError`, `GridCopyErrorReason` and `GridCopyCallbacks` types.

## PivotGrid v1.11.11 — November 18, 2026

`@kanunilabs/pivotgrid-core@1.11.11` · `@kanunilabs/pivotgrid@1.6.11` · `@kanunilabs/pivotgrid-react@1.8.11` · `@kanunilabs/pivotgrid-enterprise@1.7.11` · `@kanunilabs/pivotgrid-react-enterprise@1.8.11`

_A large, fully expanded pivot no longer freezes the page, also while rows stream in: results arrive as typed arrays and are processed in short slices, opening or closing one branch computes nothing, expand-all computes once, and live updates are spaced by their cost. A click anywhere on a collapsed group's long label opens the group again._

### Before you upgrade

- `engineResult.rowIndexMap` / `columnIndexMap` are built when first read (same content). Reading them for every cell of a large pivot is slow; use the new `rowIdOf` / `colIdOf` / `nodeIdOf` (or `PivotNode.engineId`). Copy a result with `copyResult`, not an object spread (a spread builds both maps).
- A closed branch's children can now be in `rowTree` with `isExpanded: false`: always under `'*'`, and outside `'*'` after a close that needed no compute. Read `isExpanded` (or the expanded set), not "has children", to know what is shown. `getCell` for a row inside a closed branch returns its cell, and under `'*'` colour scales and data bars count the cells of closed branches too.
- A toggle that needs no computation emits `engineCompleted` (in a later task) but calls neither `onBeforePivotCalculate` nor `onAfterPivotCalculate`. With either hook set, opening a never-computed branch runs a full compute as before.
- A live-update compute over a result already on screen no longer raises the building overlay or `isBuilding`; read `isComputing` for "a newer result is on its way". The progress bar never moves backwards. As the result grows, live updates are computed less often (about 2.5 % of the main thread); `flushAsyncTransactions()` (also with an empty queue), `applyTransaction()`, `setLoading(false)` and an export compute at once.
- Row-header cells on screen merge by member, not by equal text: two neighbouring members with equal text are two cells. Exports still merge by equal text.
- Row-header column grips moved (React and plain JS): a collapsed group's spanning label keeps only the grip of its last covered column; the grips of the columns it covers are in the corner (column-header) cells, inside their right edge (left in RTL). Code that looked for `.kanuni-pivot-resize-handle` inside a body cell by position, or counted the grips of a spanning label, now finds one; the corner cells carry one per row-header column.

### Added

- `rowIdOf`, `colIdOf`, `nodeIdOf`, `copyResult` and `PivotNode.engineId` from `@kanunilabs/pivotgrid-core`: look up a cell's engine ids without building the index maps.

### Changed

- A large, fully expanded pivot (expand all, or a saved state with `'*'`) no longer blocks the page: the worker posts the result as typed arrays, and the main thread builds the tree, sorts it (a level of 50k members included) and indexes its cells in tasks of about 24 ms. Both renderers build their rows once per result and splice a branch in or out when it opens or closes. A reply that a newer compute replaced is dropped before it is decoded.
- Under `'*'`, opening or closing one branch computes nothing (the result keeps the children of closed branches). Outside `'*'`, closing a branch and opening it again computes nothing, and opening a branch whose rows were never computed computes only that branch.
- `expandAll({ allLevels: true })` computes once: the groups are counted in the worker.
- Live updates (`applyTransactionAsync`) are spaced by the measured cost of their compute, so a slow feed into a fully expanded pivot keeps the page responsive and is not lengthened by the main thread.

### Fixed

- A collapsed group's label that spans the empty row-header columns (1.11.10) carried a column-width grip on each column's edge inside the label, so a click on most of the label began a column resize instead of opening the group. Only the label's outer edge keeps a grip now (a double-click there still fits the column), and each row-header column has its grip in the corner cell, so a column no row offers a grip for (every group collapsed) can still be resized.

## PivotGrid v1.11.10 — November 17, 2026

`@kanunilabs/pivotgrid-core@1.11.10` · `@kanunilabs/pivotgrid@1.6.10` · `@kanunilabs/pivotgrid-react@1.8.10` · `@kanunilabs/pivotgrid-enterprise@1.7.10` · `@kanunilabs/pivotgrid-react-enterprise@1.8.10`

_A `layout.virtualization` option for long tables: draw every row (`mode: false`, or `'auto'` under a threshold), set how many rows are drawn beyond the view (`overscanY`), or show row-height placeholder lines instead of a blank body while a fast scroll or a jump to the end waits for its rows (`placeholderRows`); and a `precision` field option for exact decimal totals (`precision: 2` sums a `decimal(18,2)` column to 343649.75, not 343649.74999999994); one keyboard focus (Down on a field chip enters the table, a merged row-header cell is one stop), and Ctrl/Cmd+C that works on plain HTTP, copies row headers and tells the app when it fails (`onCopy`, `onCopyError`)._

### Before you upgrade

- Nothing changes unless you set `layout.virtualization`: with no setting the body draws the same rows and the same DOM as in 1.11.9.
- Nothing changes unless a Values field sets `precision`: without it the engine adds doubles exactly as in 1.11.9, and the cells, totals, display modes, exports and selection statistics hold the same numbers.
- Down on a field chip inside the table (the Values, Columns and row-field chips) now moves the keyboard focus into the table, to the first row in view: a row field's chip to its own row-header column, any other chip to the first data column. Before, the key started the table's focus frame while the chip kept the real focus. Down on a chip of the filter strip does nothing, as before.
- Arrow keys over the row headers treat a merged cell as one cell (React and plain JS): a Region drawn once for its three cities is one stop, its first row, not three; Down leaves it from its last row. Code that pressed Down a number of times to reach a given row in a row-header column lands elsewhere now; in the data columns every row is still one step.
- With `precision` on an `avg` field, the selection statistics now add the cell averages as they are (three cells of 7/3 total 7); in the 1.11.10 preview they were rounded to the field's decimals first (6.99). A `calculateSummaryExpression` field with a `precision` and a display mode (a percentage, a running total) is no longer rounded to that precision: its own result never is.
- A `precision` that is not an integer 0 to 10 (2.5, 11) is no longer sent to a `serverSource` in the request's Values entries; the browser ignores and reports it, so the server no longer acts on it either.
- Ctrl/Cmd+C with no cells selected but text selected (in the pivot or on the page) no longer calls `onCopyError` with `'empty-selection'`: the browser copies the text and the pivot keeps out of it.
- Nothing changes unless you set `--pivot-row-height`: without it no class or variable is added and every density paints the rows it painted in 1.11.9.
- Keyboard moves in the React grid now bring a row in from below fully into view: PageDown and Down left the focused row under the pane's bottom edge by the height of the chip band above the column headers (about 120px with the Values and Columns zones), so the focus could not be seen. The pinned row-header layout only; plain JS already measured the band.
- Nothing changes unless you set `--pivot-chip-height`: unset, a chip is as tall as its content with the same 4px vertical padding as in 1.11.9 (the padding is now written as `padding-block`, computing to the same value).
- Nothing changes unless you set `--pivot-area-border`, `--pivot-area-separator` or `--pivot-area-gap`: each falls back to what its spot painted in 1.11.9 (the zones' edge `--pivot-border` at 60%, the line under the filter strip `--pivot-border`, the strip's 8px).
- Ctrl/Cmd+C now copies through the browser's copy event, and only while the focus is in the pivot or nowhere: not from a text field, not when the page's own text is selected, not while another component outside the pivot has the focus (before it copied wherever the focus was). It works on a plain-HTTP page, where it did nothing. Besides the tab-separated text it writes an HTML table (`text/html`), so a paste into Excel or Word keeps the cells. A row-header selection copies the row headers (it copied empty lines) and a data selection copies every row, the ones scrolled out of view too (they were empty). A tab or a line break inside a cell becomes a space. To hear about a copy that failed, set `onCopyError`.
- A click on a continuation row-header cell (the empty cell under a group's value, rows 2 and on of a merged group) no longer expands or collapses the group: the group's own cell and its +/- still do. A row-header selection now covers whole merged cells of its last column: a press on a continuation cell selects its group from the cell with the value, and a range that starts or ends inside a group grows to the group's first and last rows (`onSelectionChange` reports those rows). A click that collapses a group leaves the selection on the group's own row (React and plain JS).
- Nothing changes unless you set `layout.areaHeaders`: with no setting every zone draws its title row, with the same DOM as in 1.11.9.
- In the tabular row-header layout, the cell of a collapsed group (any row whose label is not in the last row-header column) now spans to the last row-header column: one `<td>` with `colspan`, its value cut only at the block's end, as the "Total: ..." rows already ran on; the empty cells it covers are no longer drawn in that row, so code that looked up `td[data-r="N"][data-c="2"]` on such a row finds nothing, and the row's `data-c` cell to use is the label's. The covered columns keep their resize grips inside the spanning cell, each on its own column's edge. An open group and the tree layout are unchanged; the column widths and the double-click fit (it does not measure a spanning value) are unchanged (React and plain JS). `onCellPrepared` and a row field's `cellTemplate` are not called for the covered row-header columns of a collapsed row: the label's own call (`columnIndex` = its depth) formats the whole spanning cell, so a `width` or `maxWidth` it sets per column now narrows the spanning cell.
- Ctrl/Cmd+C goes through the browser's `copy` event, which the pivot answers on `document` in the bubble phase: an app's `copy` listener on `window`, or on `document` added after the grid was created, runs after it, and one that calls `clipboardData.setData` there writes over the pivot's data and wins. Because a copy also writes an HTML table (`text/html`), a single copied cell pastes into Word or Outlook as a 1×1 table, not as plain text (the same in DataGrid 1.10.17); a plain-text paste (Ctrl+Shift+V) gives the text.

### Fixed

- One keyboard focus: a key pressed on a field chip inside the table (Values, Columns, a row field) that the chip did not use, Down first, went on to the table and started its focus frame while the chip kept the real focus; the next arrows then moved the chips and the frame at once, and the frame could stop on an empty row-header cell (React). The table now answers only the keys pressed on itself; a chip, a button, a field or a menu inside it keeps its own (React; plain JS already did). Down on such a chip moves the keyboard focus into the table (see above).
- The keyboard focus frame never stops on a row-header cell drawn empty: a continuation cell of a merged group is its group's first cell, the empty cells right of a collapsed or subtotal row's label are the label, and Left into a merged cell then Right out of it goes back to the row you came from, as a spreadsheet's merged cell does (React and plain JS). The subtotal label now shows the focus ring when the keyboard is on it; it was invisible there (React).
- Ctrl/Cmd+C copied nothing on a plain-HTTP page (`http://10.0.x.x`): `navigator.clipboard` exists only on HTTPS and localhost, and the failure was swallowed. The copy now goes through the browser's copy event, which every page has; `navigator.clipboard` is only a fallback for a browser that fires none (React and plain JS).
- The selection statistics rounded each cell of an `avg` field with a `precision` to that many decimals before adding (three averages of 7/3 totalled 6.99 instead of 7), and a display mode rounded a `calculateSummaryExpression` field with a `precision` (0.4567 read 0.46). Only the cells that are whole units (`sum`, `min`, `max`) are added in units now (React and plain JS).
- A Ctrl/Cmd+C with no cells selected but text selected with the mouse in the pivot copied the text, yet reported `'empty-selection'` to `onCopyError`; it reports nothing now (React and plain JS).
- Copying a selection down a long merged row-header group took time quadratic in its rows: each row walked up to its group's start. Group starts are now found once per column (React and plain JS).
- A keyboard move down (PageDown, Down) in the React grid left the focused row hidden under the pane's bottom edge: the scroll counted only the column-header rows, not the chip band above them. It now measures the header block as drawn, as plain JS does.
- `serverRequestFor` forwarded an invalid `precision` (2.5, 11) to the server; only a valid one goes now.
- Ctrl/Cmd+C of selected row-header cells copied empty lines (React; plain JS left a merged cell's other rows empty), and of data cells scrolled out of view, empty values. The copy now reads what the cells show, from the result when a cell is not drawn; a merged row-header cell gives its value on every row it spans, so each line carries its full path for sorting and filtering in a spreadsheet, and a row-header selection that ends on a merged cell copies all its rows. The expand/collapse glyph is not copied.
- A click on a continuation row-header cell collapsed the group and the selection then fell on another row's cell; a range dragged or Shift-extended from such a cell painted the continuation cells as a bare selection block without the group's value. The cell is now its merged group, as for the keyboard: the focus and the selection go to the group's first cell, the selection covers the whole group with its value painted at the top of one block, and nothing collapses; a double click there toggles nothing either (React and plain JS).
- A collapsed group's value in the row headers was cut at its own column's width ("REEL TE...") while the row-header columns right of it stayed empty in that row. Its cell now runs over them to the end of the row-header block and the value is cut, by `textOverflow`, only there; an open group stays in its own column, and the rows stay level with the data in the pinned, unpinned and RTL layouts (React and plain JS).

### Added

- `layout.virtualization: { mode?: boolean | 'auto'; threshold?: number; placeholderRows?: boolean; overscanY?: number }` (React and plain JS). `mode` `true` (default) draws the rows in view plus a margin, as before; `false` draws every row at once; `'auto'` draws every row while there are at most `threshold` (default 2000) and a window above that, counting the rows the table shows (subtotal and grand-total rows included), not the source rows. Drawing every row of a large table takes seconds and freezes the page meanwhile, again on every expand, collapse, sort or filter: keep `false` to small tables. `placeholderRows: true` (default `false`) fills the part of the body whose rows are not drawn yet with a line at every row height, in the row headers and the data alike, instead of a blank area; colours `--pivot-placeholder-row-line` (default `--pivot-border`) and `--pivot-placeholder-row-bg` (default transparent). `overscanY` is the number of rows drawn beyond the view, above and below alike, a part-shown row counting as in view; a `threshold` or `overscanY` that is not a finite number of 0 or more is ignored; not set, the margin is the one of before (about 5 rows each side in React, 5 above and 10 below in plain JS). Rows only: past 40 data columns the columns are still drawn in a window, so while every row is drawn each sideways scroll step redraws every row. It can change on a live grid (`setLayout`, a new `layout` prop): the body is redrawn, the data is not recomputed. A view setting and the app's: not saved with the state, and a restored state keeps the grid's current value.
- `pivotRowVirtualization(layout, rowCount)`, `PIVOT_VIRTUALIZATION_THRESHOLD` and `PLACEHOLDER_ROWS_CLASS` are exported from `@kanunilabs/pivotgrid-core`, and the `PivotVirtualizationOptions` type from `@kanunilabs/pivotgrid-core`, `@kanunilabs/pivotgrid` and `@kanunilabs/pivotgrid-react`.
- `PivotField.precision` (an integer 0 to 10; React, plain JS and both Enterprise wrappers): the decimal places of a Values field's source values, for exact totals. Thousands of two-decimal values added as doubles can total 343649.74999999994, which a one-decimal format writes as '343.649,7' in `tr-TR` where the decimal total 343649.75 reads '343.649,8'. With `precision: 2` each value is rounded once to two decimals (half away from zero) and the engine adds whole cents, so every cell, subtotal and grand total is the number nearest the decimal result. Applies to `sum`, `avg` (exact sum divided by the count), `min` and `max`; `count`, `distinct` and `custom` are unchanged. The display modes read the exact totals (a running total or a difference is added in the same units, a percentage divides two exact totals), as do the top-N 'Others' row, summary expressions (their own result is not rounded) and the exports, which take the engine's numbers: Excel gets 343649.75 and its number format rounds as the screen does. A row-level `calculateExpression` result is rounded like a source value. Totals are exact while they stay under `Number.MAX_SAFE_INTEGER / 10^precision` (about 90 trillion at 2); past that they are added as doubles, as without the setting, never NaN. Another value (2.5, -1, 11) is ignored and reported through `onError`. With a `serverSource` the browser does not add: each Values entry of the request carries `precision` and the server sums (`rollupInMemory` honours it).
- The selection statistics (`onSelectionStatsChange`, the footer) add the selected cells in the same units when every selected cell's field has a `precision`, so they agree with the table; a selection with any cell of a field without one adds doubles as before (React and plain JS). `ExactSum`, `precisionScale` and `MAX_PRECISION` are exported from `@kanunilabs/pivotgrid-core`.
- `onCopy(event)` and `onCopyError(error, { reason })` (React props, plain JS config; both Enterprise wrappers pass them on). `onCopy` gets `{ text, html, rows, cols }` after a Ctrl/Cmd+C wrote the selection. `onCopyError` gets a `PivotCopyError` and its `reason`: `'insecure-context'` (no copy event and no clipboard: plain HTTP), `'permission-denied'` (the browser refused the write), `'empty-selection'` (Ctrl/Cmd+C with the focus in the pivot and nothing selected) or `'unsupported'`. The grid shows nothing itself: tell the user from here. `bindPivotCopy`, `copyPayload`, `cellCopyText`, `pivotOwnsCopy`, `writeCopyEvent`, `writeClipboardFallback` and `pivotCopyError` are exported from `@kanunilabs/pivotgrid-core`, with the `PivotCopyInfo`, `PivotCopyError`, `PivotCopyErrorReason` and `CopyBindingOptions` types.
- `--pivot-row-height` (any CSS length, 16 to 200px; none by default; React and plain JS): the height of every body, subtotal, grand-total and column-header row, over the `density`. Delphi's 21px rows are `--pivot-row-height: 21px`. The renderers read it (rounded to whole px) and run the virtual window, the spacers, the placeholder lines and the keyboard scrolling on the same number, and the line heights follow it (a `kanuni-pivot-custom-row-height` class and `--pivot-row-h` on the root in plain JS, on the body in React), so the frozen row headers stay level with the data, under CSS zoom too. Read when the grid is created, on a resize and when the root's class, `style` or theme changes. `readPivotRowHeightVar`, `applyPivotRowHeightVar`, `watchPivotRowHeightVar`, `PIVOT_CUSTOM_ROW_HEIGHT_CLASS`, `PIVOT_ROW_HEIGHT_RESOLVED_VAR` and `PIVOT_ROW_HEIGHT_RANGE` are exported from `@kanunilabs/pivotgrid-core`.
- `--pivot-chip-height` (any CSS length; none by default; React and plain JS): the height of every field chip, in the filter, Values and Columns zones, the row-header corner and the field list (its host carries the root's class). Delphi's ~22px chips are `--pivot-chip-height: 22px`; the vertical padding shrinks around the 12px text. On a touch screen (or with `density: 'touch'`) every chip stays at least 44px tall, with or without its menu, so its 44px grip and buttons stay inside it. The drag ghost, a larger floating label, keeps its size.
- CSS variables, none defined by default (React and plain JS): `--pivot-area-border` (the edge of the filter, Values and Columns zones; it was `--pivot-border` at 60%, tied to the grid lines' colour), `--pivot-area-separator` (the line under the filter strip, between it and the Values zone; `--pivot-border`) and `--pivot-area-gap` (the space around the filter zone; 8px). With `--pivot-area-bg` and `--pivot-area-header-bg` (1.11.9) the zones can be drawn apart from the grid, as a desktop pivot's light separator is.
- `cellUnitScale(field)` (the scale a field's finished cells are whole units of: 0 for `avg` and summary expressions) and `rowHeaderCellLookup(rows, headerCols, tree)` (`rowHeaderCellAt` for many cells, each group walked once) are exported from `@kanunilabs/pivotgrid-core`.
- `pivotFocusMove`, `rowHeaderCellAt`, `rowHeaderGroupLastRow` and `isRowHeaderContinuation` (and the `RowHeaderNavRow` type) are exported from `@kanunilabs/pivotgrid-core`: the keyboard rule over merged row-header cells both renderers use.
- `rowHeaderSelectionRows(rows, minR, maxR, col, headerCols, tree)` is exported from `@kanunilabs/pivotgrid-core`: the rows a row-header selection covers, grown to whole merged cells of a column.
- `layout.areaHeaders: { filter?: boolean; data?: boolean; column?: boolean }` (React and plain JS; each `true` by default): `false` leaves that field zone without its title row, the "FILTER AREA", "VALUES" or "COLUMNS" line with its icon, and on the filter strip its collapse button and the count of filtering fields. The chips, the empty zone's hint and the drop target stay where they were and the zone is as tall as its chips; it keeps its name for screen readers (`role="group"` with an `aria-label`). Without the title row the filter strip collapses only through `filterAreaCollapsed`, from the app's own button. Collapsed with `filterAreaCollapsedRows: 0` and no title row, the strip has nothing to show and is not drawn, not even during a drag. A view setting and the app's: not saved with the state, and a restored state keeps the grid's current value. `areaHeaderShown(layout, area)` and `filterStripEmpty(layout)` are exported from `@kanunilabs/pivotgrid-core`.
- `rowHeaderLabelSpan(row, headerCols, tree)` is exported from `@kanunilabs/pivotgrid-core`: how many row-header columns a row's label cell spans.

## PivotGrid v1.11.9 — November 17, 2026

`@kanunilabs/pivotgrid-core@1.11.9` · `@kanunilabs/pivotgrid@1.6.9` · `@kanunilabs/pivotgrid-react@1.8.9` · `@kanunilabs/pivotgrid-enterprise@1.7.9` · `@kanunilabs/pivotgrid-react-enterprise@1.8.9`

_Ten new theme variables for the selected cell, the zone and corner surfaces, the Grand Total header, the data ground and the field chips, and a `filterPopupSearch` layout option for a filter list without a search box; the filter popup now grows to the edge of the screen and remembers its size, `layout.viewOptionsMenu` lets the app decide which items the View options menu lists, a date field's filter list reads as its headers do, the whole Columns strip over the data columns takes a dropped field, and a blank row or column value reads as the dictionary's `blanks` text in every language ('' allowed)._

### Before you upgrade

- Nothing changes unless you set one of the new variables or `layout.filterPopupSearch`: each variable falls back to the expression its spot used in 1.11.8, so the default look is the same colour for colour.
- A date field's filter list (React and plain JS) now writes each date as its header does, not as the ISO text the data holds: the field's `format` when it is a date format (`{ day: '2-digit', month: '2-digit', year: 'numeric' }` gives 31.03.2026 in `tr-TR`), the grid's default date text without one, and a month or weekday `groupInterval` by name. Times of one day are one row. The list no longer reads a `format` that is not a date format on a date field (a function, number options): it writes what the header writes (with a month `groupInterval`, 'March', not '3'). Its order, its ticks and the stored filter are still the raw values, so a saved filter keeps working, and the search box finds the text on screen.
- A date field whose `format` holds date options (`day`, `month`, `year`, `weekday`, `hour`, `dateStyle`...) is now written with them in its row and column headers; before the options were ignored there and the header read the default long date. A field with no such format, a number format, or a `groupInterval` reads as before.
- The header filter popup can be dragged larger than 350 x 600 px, up to the edge of the visible area; it still opens at 250 x 350 px. Lower the limit with `--pivot-filter-popup-max-height` / `--pivot-filter-popup-max-width` if a taller popup is not wanted.
- Blank values are now written with the dictionary's `blanks` text in every language: in row and column headers, subtotal labels, the filter list, the pivot's Excel, PDF, print and clipboard headers, chart categories and the drill-down title (React and plain JS). A Turkish grid shows '(Boşlar)' where it showed '(Blank)', a German one '(Leer)', and so on; the English dictionary's `blanks` is now '(Blank)' (it was '(Blanks)'), so an English grid looks as before except where '(Blanks)' was shown: the filter list's rare null row and the Enterprise chart's blank category. To keep the old text in every language, pass `dictionary: { blanks: '(Blank)' }`. `blanks: ''` leaves the header cell empty and the filter list row empty but tickable, named for screen readers by the language's own blanks text. The internal member is still '(Blank)': filters, path keys, saved state, `rowPath` / `colPath` and node values are unchanged, so saved filters open as they were. A row header's `cellTemplate` gets the shown text (the blanks text, a B.95 date format) as both `value` and `displayValue`, in React and plain JS, as it already got the formatted text; a column header's `cellTemplate` gets the raw member ('(Blank)') as `value` and the shown text as `displayValue`. The list export (source rows) and the CSV of raw data are unchanged.
- The Columns strip over the data columns is a drop target along its whole visible width, not only where the Columns box is drawn (React and plain JS): a field dropped on the strip past the box goes to the END of the Columns area, as the box's own empty end did. Inside the box nothing changed (before or after the nearest chip). The box keeps its look; while a field is dragged into the column area the strip is tinted (`.kanuni-pivot-column-strip[data-drop-over]`, a 7% `--pivot-primary` wash).

### Changed

- The header filter popup's size limit is the visible area (from the popup's position to the screen edge) instead of a fixed 350 px high and 600 px wide; the opening size is unchanged (React and plain JS). The virtual list draws the rows the new height shows.
- A date field's filter list writes dates as its headers do (see above); a date field's `format` with date options is now used by its headers too (React and plain JS). Charts and exports keep their own date text.
- Blank row and column values read as the dictionary's `blanks` text, in every language and on every surface that shows them (see above); the English `blanks` text is '(Blank)'. An app's `dictionary.blanks` wins, and '' is allowed: an empty header cell, an empty list row with the language's text as its accessible name; a search does not match that empty row.
- The whole Columns strip over the data columns takes a dropped field: past the box it goes to the end of the area (see above). The strip cell carries `data-drop-id="column-strip"` and the class `kanuni-pivot-column-strip`; the Columns box keeps `data-drop-id="column"`.

### Fixed

- A column field member `0` or `false` was captioned '(Blank)' in the column header and its subtotal label (a truthiness test); it reads `0` / `false` now (React and plain JS, and the pivot exports).

### Added

- `layout.viewOptionsMenu: { filterArea?: boolean; footer?: boolean }` (default both `true`; React and plain JS): chooses which items the View options menu (right-click on a data cell) lists. `footer: false` leaves out Footer bar, which keeps an app that sets `showFooter: false` from having the user bring it back; `filterArea: false` leaves out Filter area; with both `false` the menu does not open. Items that stay still write `showFooter` / `showFilterArea` into the layout and `onLayoutChange`. A view setting. It is the app's setting, not the reader's: `viewOptionsMenu` and `filterPopupSearch` are not written into `getPersistedState`, `stateStoring` or a saved report, and a restored state (an asynchronous `onLoad`, `applyPersistedState`, a report) keeps the grid's current value, ignoring one an older state may hold. While an item is left out, the reader cannot switch that bar, so a restored state keeps the grid's current `showFooter` (or `showFilterArea`) too; a listed item's saved value is restored as before.
- `--pivot-filter-popup-max-height` and `--pivot-filter-popup-max-width` (any CSS length, none defined by default): lower how far the header filter popup can be dragged. By default it grows to the edge of the visible area, zoom and on-screen keyboard included.
- The header filter popup remembers the size a user drags it to, per field, for the life of the page (kept in memory, not stored), and opens at that size next time, fitted to the screen.
- CSS variables, none defined by default (React and plain JS): `--pivot-cell-selected-bg` and `--pivot-cell-selected-text` (a selected cell), `--pivot-area-bg` and `--pivot-area-header-bg` (the body and the title row of the filter, Values and Columns zones; dialog titles stay on `--pivot-surface-header`), `--pivot-header-total-bg` (the Grand Total column header and the Grand Total label of the grand-total row), `--pivot-corner-bg` (the corner cells over the row headers), `--pivot-cell-bg` (the data cell's own ground, under totals, conditional formats and anything `onCellPrepared` sets), `--pivot-row-area-empty-bg` (the band under the row headers when the rows are few; React), `--pivot-chip-font-size`, and `--pivot-chip-border-light` / `--pivot-chip-border-dark` (a raised chip edge: top and left light, bottom and right dark). Set them on `.kanuni-pivot-root`; the field list and drag ghost host carries the same class. Listed with their defaults on the theming page.
- `layout.filterPopupSearch` (default `true`; React and plain JS): `false` leaves the search box and the type-ahead out of the header filter popup, for a plain list; Select All, Clear All, the keyboard keys and the grouping of rows by text are unchanged. A view setting.
- `chartIntegrationOptions.blanksText` (React): what the chart calls a blank category. The grid fills it from its dictionary's `blanks` unless the app sets it; `ChartDataTransformer.transform` reads it, and a transformer of your own receives it.
- `BLANK_MEMBER` ('(Blank)', the engine's member for a missing value), `isBlankMember(value)` and `blanksTextOf(dictionary)` are exported from `@kanunilabs/pivotgrid-core`; `formatDimensionValue` and `formatHeaderValue` take an optional blanks text as their last argument (omitted, they return what they did before).

## PivotGrid v1.11.8 — November 16, 2026

`@kanunilabs/pivotgrid-core@1.11.8` · `@kanunilabs/pivotgrid@1.6.8` · `@kanunilabs/pivotgrid-react@1.8.8` · `@kanunilabs/pivotgrid-enterprise@1.7.8` · `@kanunilabs/pivotgrid-react-enterprise@1.8.8`

_An opt-in `groupByFormat` for a row or column field: values that show the same formatted text are one row or column, and the header writes that text, as the filter list already does._

### Before you upgrade

- Nothing changes unless a field sets `groupByFormat: true`; without it grouping, headers, sorting, paths and exports are as in 1.11.7.
- With `groupByFormat: true` a member of that field is its formatted text, not the raw value: `rowPath` / `colPath` in `cellClick`, `getCellRecords`, `expandedPaths`, `sortBySummaryPath`, a node's `value` and a `calculateCustomSort` comparator all receive the text ("18,2"). Code that filters to one member from such a path should pass `controller.memberFilterValues(fieldId, member)` to `applyFilter` instead of `new Set([member])`, since a filter names raw values.

### Added

- `PivotField.groupByFormat` (default `false`; React and plain JS, Community): a number field with a `format` in Rows or Columns is grouped by the text that format writes in the grid's `locale`, the same text its filter list shows. Two-decimal data shown with one decimal (18.21, 18.23, 18.24 as "18,2" in `tr-TR`) is one row whose cells sum every raw value under it; the header, the filter list, exports (Excel, PDF, print, clipboard), charts, `cellClick` records, the drill-down and the selection detail all use that text and that group. Members sort by their smallest raw value, not alphabetically, with blanks last. `-0` and `0` are one member, written as `0` (the filter list of a formatted number field now lists them as one `0` row too; before, Intl wrote a separate `-0`). The field's filter still stores raw values. The option does nothing without a `format`, with a `groupInterval` (which already groups), on a date field, in the Values area, and with a `serverSource` (ignored in the browser; the server groups: your own `serverSource.compute` sees the flag on the field objects, the request `pivotServerSource` sends does not carry it). Each value is formatted once per distinct value on the main thread while the data is encoded, so the worker only sees the texts and a `format` function works. Turning the option on or off on a field drops that field's open branches, as moving it does; a saved state written with the option on reopens its branches while the option stays on.
- `controller.memberFilterValues(fieldId, member)`: the filter values that keep exactly one member, `new Set([member])` for an ordinary field and every raw value under the text for a field with `groupByFormat`. The plain JS Enterprise chart cross-filter uses it, so clicking the "18,2" bar filters to its raw values.
- `groupsByFormat(field)`, `groupKeyOf(row, field, locale)` and `formattedGroupKey(value, field, locale)` are exported from `@kanunilabs/pivotgrid-core` for code that matches source rows to such members.

## PivotGrid v1.11.7 — November 16, 2026

`@kanunilabs/pivotgrid-core@1.11.7` · `@kanunilabs/pivotgrid@1.6.7` · `@kanunilabs/pivotgrid-react@1.8.7` · `@kanunilabs/pivotgrid-enterprise@1.7.7` · `@kanunilabs/pivotgrid-react-enterprise@1.8.7`

_A third value for the Values and Columns zones: `layout.areaChipOverflow: 'shrink'` keeps the chips on one line by narrowing them all, and scrolls only when they do not fit even at the floor._

### Before you upgrade

- A filter on a field in the Values area (picked on its chip, or set with `applyFilter` / `filterValues`, also in a saved report) now filters the source rows, so cells and totals change. Before, the table ignored it while the drill-down, exports and charts applied it. A report saved with such a filter therefore shows different, filtered numbers after the upgrade; clear the filter on the chip if the old totals were the intended ones.
- In the filter list of a number field with a `format`, values that show the same text are now one row (two-decimal data shown with one decimal: "18.2" once, not once per raw value), and the row ticks or unticks every raw value under it. The stored filter is unchanged: it still lists raw values, so a saved filter that names only some of them opens with that row half-ticked (`aria-checked="mixed"`). Code that found a list row by a raw value other than the smallest in its group no longer finds it.

### Changed

- The header and chip filter list (React and plain JS) groups the values of a field by the text it shows: with a number `format`, raw values that round to the same text ("18.21" and "18.24" shown as "18.2") are one row, placed where its smallest value sorts, and ticking it filters all of them. Select All, Clear All and search work on these rows; a row with only some of its values ticked shows a half tick. The filter itself still stores raw values, and fields without a format (text, dates, date groups) list every value as before. A `format` given as a function now labels the list too, as it labels the cells; before, the list built a default number format from it (at most three decimals), so values the cells show apart could share a row.

### Fixed

- A filter on a number field in the Values area did nothing to the table: the chip showed the filter mark, but cells and totals stayed unfiltered, and the drill-down (`getCellRecords`) and exports, which did apply it, disagreed with the table. The filter now filters the source rows before they are summed, wherever the field sits, by its raw value: a blank is not a zero, `-0` is `0`, and `0.1 + 0.2` is not `0.3`. Table, totals, drill-down, exports, charts, live updates and a server source give the same rows. A field moved between the row or column area and Values keeps its filter.

### Added

- `layout.areaChipOverflow: 'shrink'` (React and plain JS): the Values and Columns zones stay on one line, with no scrollbar, and the chips narrow in proportion until they fit. The caption is clipped (no ellipsis, whatever `layout.textOverflow` is) and the chip's tooltip keeps the full name; the sort arrow, the badge and the buttons keep their size. The floor is 56 px (`--pivot-chip-shrink-min`, never under 44 px with touch chips); only when the chips do not fit at the floor does the line scroll sideways. Dragging, the drop mark and touch work as in `'scroll'`. The default (`'wrap'`) is unchanged. A chip at its floor clips its badge and buttons with `overflow: clip` and an 8 px `overflow-clip-margin` for the drop mark beside it; in a browser without `overflow-clip-margin` (Safari, per caniuse; not tried there) that mark can be cut off at the chip's edge.

## PivotGrid v1.11.6 — November 16, 2026

`@kanunilabs/pivotgrid-core@1.11.6` · `@kanunilabs/pivotgrid@1.6.6` · `@kanunilabs/pivotgrid-react@1.8.6` · `@kanunilabs/pivotgrid-enterprise@1.7.6` · `@kanunilabs/pivotgrid-react-enterprise@1.8.6`

_Round 5 of customer requests: the drag ghost above the field list window, a resize guide line that stays under the pointer when the pivot is scrolled or zoomed, a double click on a column edge that fits the column, a text clip setting, font-weight variables, a context menu that ends at the pointer, no focus flash on the first cell; then lines at the end of every row-header group (opt-in), Values and Columns zones on one scrolling line (opt-in), a drop between two row-header chips that lands between them, an unpinned table with its vertical scrollbar at the pivot's edge, data rows kept in view in a short, zoomed pivot, and the focus kept in the pivot after an undo from the data pane._

### Before you upgrade

- The drag ghost of a field is now drawn in the same host in <body> as the field list window and the menus (it was a child of the pivot), above all of them at z-index 100000, and only while a field is dragged. A style that reached it as a descendant of the pivot (`.my-grid .kanuni-pivot-drag-overlay`) no longer matches; the class `kanuni-pivot-drag-overlay` itself is unchanged.
- The guide line of a column resize follows the pointer when the row headers are unpinned and scrolled sideways, and under CSS `zoom`: the width a drag commits is now what the pointer moved in the pivot's own pixels. Before, at a zoom of 150% a drag of 40 pixels made the column 60 pixels wider. Saved widths are unchanged.
- A double click on the edge of a column header (the resize grip) now fits that column to what is rendered: the header and the visible cells. It commits through `onColumnWidthsChange` (React) or the controller (`columnWidths`) like a drag. Before, it did nothing. Rows the virtual window has not rendered are not measured.
- A right-click menu (the row and column header menu, the view-options menu) opened near the bottom of the window now ends at the pointer when it opens upward. Before, it ended up to 107 px above it, because its height was assumed to be 200 px.
- A mouse click on a cell no longer puts the focus ring on the first cell for an instant before it moves to the clicked one. Focus that arrives with the Tab key still starts at the first cell.
- With the row headers unpinned (`layout.rowHeaderOverflow: 'unpin'`, or unpinned on their own when they cannot fit) the React grid now scrolls in ONE element both ways: the wrapper that holds the row headers and the data (class `kanuni-pivot-unpinned-scroller`, the parent of `.pivot-right-pane`). Its vertical scrollbar is at the pivot's right edge; before, `.pivot-right-pane` scrolled down by itself and, as wide as its data, put its scrollbar after the last data column. `.pivot-right-pane` no longer scrolls in this mode: code that reads or sets its `scrollTop` must use the wrapper. The scroll saved with `getPersistedState({ include: ['scroll'] })` is the wrapper's in this mode, so its left offset now includes the row headers. Pinned, `'shrink'` and `'scroll'` are unchanged, and so is the plain JS grid (one scroller already).
- A field dropped on a tabular row-header cell (the space around a row chip above the rows) now goes before or after that cell's field, split at the cell's middle, with the chip's insertion mark. Before, the space between two row chips was the end of the row area: the field became the last row field and no mark showed (in the plain JS grid the cells were not a target at all). The empty end of the last cell still appends, since after the last field is the end.
- A narrow pivot (768 px or less, the pivot's own width) that is shorter than 480 px now keeps the filter area to 44 px and the Values and Columns areas above the headers to 40 px (60 and 56 px with touch chips), one row of chips each, scrolling inside. Before, a 1100x500 window at 175% left a 240 px pivot no data row in view. A pivot of 480 px or more, and every wider pivot, is unchanged. The wrapper carries `data-pivot-short` while this applies.
- After `undoLayout()` / `redoLayout()`, when the element that had the focus is still there but no longer takes it (the data pane, which a browser focuses only while it can scroll, after a step that shortened the table), the focus now goes to the grid element; the focus ring then starts at the first cell, as after a Tab. Before, the focus stayed on <body> and the next Ctrl+Y reached nothing.
- Row-header cells now carry `data-group-start` (the first row of a group, where the value is written) and `data-group-end` (the last row of the group in that column), whether or not `layout.rowHeaderGroupLines` is on. They change no style; a test that compares the cells' markup will see them.
- With the focus on the grid but on no cell (a click below the last row, on the header band, or on the empty part of an unpinned table), the first arrow, Home, End, PageUp or PageDown key now puts the focus on the first cell, selects it as a plain move would, and is consumed (its default is prevented). Since the mouse no longer focuses the first cell (B.85), these keys did nothing there.
- `onCellPrepared` gets the plain font weights it always got (600 for a total, 500 for a column header). The cell is then painted with the variable that defaults to that weight (`var(--pivot-font-weight-total, 600)` and its siblings) unless the hook set another value: code that reads a cell's inline `font-weight` back from the DOM sees the `var(...)` text.
- In the plain JS grid the field list window and the drag ghost now carry the root's view classes and variables (`showSummaryBadge: false`, `showChipRemove: false`, `textOverflow: 'clip'`, the density, `scrollbarStyle: 'native'`, the chip width) and follow a layout change while open. Before, they kept the default look in <body>; the React grid has done this since 1.11.5.

### Added

- `layout.textOverflow` (default `'ellipsis'`, as before): `'clip'` cuts the text of every field chip (the strips, the field list, the drag ghost) and of the row-header values ("Total: ..." included) at the edge instead of ending it with "...". Number cells do not change. In `'clip'` mode a row-header value carries its whole text as a tooltip. React and plain JS.
- CSS variables for the font weights, each defaulting to the weight used until now: `--pivot-font-weight-chip` (500), `--pivot-font-weight-group` (600, the row-header values that have a level below), `--pivot-font-weight-total` (600, subtotal and total cells and labels), `--pivot-font-weight-grand-total` (700), `--pivot-font-weight-header` (500, the column headers) and `--pivot-font-weight-header-total` (600, the total column headers). Set them on the page or above the pivot; the screen only, the exports do not change.
- A double click on a column's resize grip fits the column to its content (row-header columns, data columns and column groups). React and plain JS.
- `layout.rowHeaderGroupLines` (default `false`, as before): a 1px line at the end of every row-header group in every row-header column, also where a group of several rows ends on a continuation cell and between two subtotal rows in a row (where the row-header block drew none). It is the same bottom border a one-row group already has, so no row changes height (measured in Chromium, Firefox and WebKit, pinned, unpinned and tree). With it on, a border `onCellPrepared` gives such a cell is kept. React and plain JS.
- `layout.areaChipOverflow` (default `'wrap'`, as before): `'scroll'` keeps the Values and Columns zones above the table on one line that scrolls sideways instead of starting a second line. Dragging to the zone's edge scrolls it, and a quick swipe on touch scrolls it. The filter area keeps its own row cap (`filterAreaMaxRows`). React and plain JS.

### Fixed

- The drag ghost went under the field list window (it was drawn at z-index 999 inside the pivot).
- The guide line of a column resize was not under the pointer when the pivot was scrolled sideways (row headers unpinned): it sat as far left as the scroll offset.
- Under CSS `zoom` the guide line of a column resize and the committed width were off by the zoom factor (a drag of 40 pixels at 150% made the column 60 pixels wider).
- The row-header context menu, and the view-options menu, opened near the bottom of the window ended above the pointer instead of at it, because they were placed from an assumed size.
- A click on a cell flashed the focus ring on the first cell before it moved to the clicked cell.
- After an undo or redo started with the focus on the data pane (a click on a cell focuses it), the focus fell to <body> in Chrome and Firefox when the step shortened the table, and the next Ctrl+Y did nothing.
- A field dropped between two row-header chips went to the end of the row area with no insertion mark; it now lands between them.
- With the row headers unpinned, the vertical scrollbar of a table wider than the pivot was after the last data column, out of view; it is now at the pivot's right edge, and the row headers, the data and the sticky header rows scroll together.
- In a short, narrow pivot (a zoomed desktop window) the filter area and the Values and Columns areas left no data row in view.
- Plain JS: the field list window and the drag ghost ignored the chip and view settings (no-summary-badge, no-chip-remove, clip, density, native scrollbars).
- After a click on no cell, the arrow keys did nothing (since the first-cell focus flash was removed).
- Unpinned: the spinner and percentage shown while the pivot is prepared sat in the middle of the whole table, out of view after a scroll down; a saved scroll restored before the grid unpinned on its own was lost; a wrapper of height 0 that was still shown drew every row.
- A short pivot could log a ResizeObserver loop warning when its height came from its content, and showed the full-size header parts for one frame.

## PivotGrid v1.11.5 — November 15, 2026

`@kanunilabs/pivotgrid-core@1.11.5` · `@kanunilabs/pivotgrid@1.6.5` · `@kanunilabs/pivotgrid-react@1.8.5` · `@kanunilabs/pivotgrid-enterprise@1.7.5` · `@kanunilabs/pivotgrid-react-enterprise@1.8.5`

_Chip settings (`showSummaryBadge`, `chipMaxWidth`, `showChipRemove`), a filter area that collapses to its header, row headers that keep their width, a single branch that can be closed after expand-all on a big pivot, a Grand Total heading over measure columns, and the browser's own scrollbars on request. Also: a failed export no longer reports success, an Excel list with readable column widths, exports that say what the grid says for subtotals and grand totals, and two fixes that 1.11.4 did not finish: focus after undo on a big dataset, and the first data row on a phone._

### Before you upgrade

- On a screen 768 px wide or narrower the filter area now stops at 30% of the pivot's height (it was 40% in 1.11.4), and the Values and Columns areas above the headers stop at 88 px and scroll inside themselves. Before, they grew with their chips and, in a 400-450 px pivot, left the first data row below the visible part of the table.
- `exportData` rejects when nothing was written: a styled format (`pdf`, `excel-list`, `excel-pivot`) with no Enterprise exporter registered, and a format nothing handles. It used to log a notice (or nothing) and resolve. The footer and menu buttons send the error to `onError` as before (an `AbortError` stays silent). The React ref's `exportData` now also rejects after calling `onError`, so an app that awaits it can tell; an app that calls it without `await` and without a `catch` will see an unhandled rejection in that case. `onError` is called and the promise rejects: rely on one of the two, not both.
- A tooltip with the whole caption now shows on a field chip.
- After `expandAll({ allLevels: true })` on a pivot of more than 5,000 groups, the entries in `expandedPaths` and `expandedColPaths` that start with U+0001 are closed branches. They are saved with the state (`getPersistedState()`, saved reports) and are opaque: do not parse them. A `serverSource` and a custom exporter are given the sets without them (the `*` entry means everything open), and a json-snapshot export leaves them out.
- In the plain JS grid a swipe over a field chip now scrolls the capped filter area and the Values and Columns areas (a hold of 200 ms, then a move, still drags), as in React.
- `undoLayout()` and `redoLayout()` watch the pivot's DOM for 10 seconds (it was 3 seconds, and stopped 100 ms after the result) and give the focus back whenever the focused element is replaced. Focus the reader moves with a click is left alone.
- The Excel files (`excel-list`, the summary table written by `exportSummaryTable`, and `excel-pivot`) now size each column from its heading and its first 1,000 rows, between 8 and 60 characters wide, reading a date-time or custom decimals from the number format. Before, a list column kept Excel's default width (long headers were cut, amounts could show as ####) and every pivot column was 15 wide. A merged heading does not set a width. A file over the streaming limit (100,000 list rows, 150,000 pivot cells; written as .xls) has no widths, as before.
- The totals in a file say what the grid says. The grid now passes its dictionary (the locale's words and your `dictionary` overrides) to the controller, and the Excel list, pivot and streaming files, the PDF, print and clipboard take the subtotal and grand-total words from it. A custom `dictionary.subTotal` of 'Toplam' used to appear on screen as 'Toplam: X' and in the Excel file as the locale's 'Alt Toplam: X'; now both say 'Toplam: X'. Print and clipboard, which wrote an English 'Total' and 'Grand Total' in every language, now follow the grid's language (in an English grid a subtotal reads 'Sub Total: A' where it read 'Total: A', so an app that parses the printed or copied text for 'Total:' must change), and the React PDF does too where the PDF font can draw the word (otherwise it keeps the English one, as the plain JS PDF does). A controller used with no renderer has no dictionary: Excel and PDF use the words of its `locale`, print and clipboard keep the English ones.
- With many row fields on a narrow or zoomed screen the row-header block now lets go of its pin by itself, whatever `layout.rowHeaderOverflow` says: when its columns at the 56 px floor plus 160 px for the data are wider than the pivot (columns x 56 + 160 > the pivot's width), the whole table (headers and data) scrolls sideways as one, each column at its width. Before, the block filled the pivot and the data columns could not be reached. Four row fields in a 358 px phone box now take this path (they were squeezed to 56 px columns); where the block fits nothing changes.
- The field list window of the React pivot is drawn in `<body>` (it was inside the pivot's box), inside a `.kanuni-pivot-portal-container`. CSS that reached it as a descendant of `.kanuni-pivot-wrapper` no longer matches (a page-wide selector still does). The host carries the pivot root's classes (`striped`, density, `no-summary-badge`, `no-chip-remove`, native scrollbars), its `--pivot-*` variables and its `dir`, so the window looks and reads as before. Every portal host, React and plain JS, now sets `zoom` on itself to cancel a CSS `zoom` on `<body>` or an ancestor, so popups are sized and placed in viewport pixels again.
- The 768 px limit of B.61 (the filter area at 30% of the pivot's height, the Values and Columns areas at 88 px) follows the pivot's own width now, not the window's: `.kanuni-pivot-wrapper` is a size container (`container-type: inline-size`) and the rules sit in an `@container` query. A pivot narrower than 768 px in a wide window gets the caps (a zoomed-in desktop); a wide pivot in a narrow window no longer does. A browser without container queries (before Chrome 105, Safari 16, Firefox 110) does not apply the caps. Because the width of `.kanuni-pivot-wrapper` is now contained, a wrapper whose width depends on its contents (an inline-block, or an auto-width column or flex item around the pivot) can collapse to 0: give the pivot's container a width, as the 100% the wrapper has by default.

### Added

- `layout.showSummaryBadge` (default `true`): `false` hides the summary-type badge ("Sum") on the value chips. `layout.chipMaxWidth` (default `120`): the widest a chip's caption grows, in px; `'none'` keeps the whole text, `'auto'` sizes to the caption up to 280 px. The whole caption is the chip's tooltip. React and plain JS.
- `layout.filterAreaCollapsedRows` (default `1`): how many rows of chips the collapsed filter area shows. `0` leaves only its header: the name, how many fields filter, and the button that opens it.
- `layout.rowHeaderOverflow: 'scroll'` (default `'shrink'`): the row-header columns keep their width instead of being squeezed toward 56 px. React holds the pinned block to 60% of the width and scrolls it sideways inside itself; plain JS unpins the row headers and the whole table scrolls.
- A single branch can be closed after `expandAll({ allLevels: true })` on a pivot of more than 5,000 groups. The closed branches are saved beside the `'*'` sentinel, with the state, in the layout undo and in saved reports.
- `layout.showChipRemove` (default `true`): `false` hides the remove (x) button on the field chips of the strips (the field list's remove button and the delete buttons of the calculated-field and pre-filter windows stay); a field still leaves by dragging it to the field list, by Delete on the focused chip or from its menu. The space between a chip's caption and its filter icon is the CSS variable `--pivot-chip-filter-gap` (default `4px`), set on the pivot or above it. React and plain JS.
- `layout.showGrandTotalHeader` (default `false`): while no field is in the column area, a "Grand Total" heading (`dictionary.grandTotal`) sits above the measure headers. The Excel pivot, PDF, print and clipboard files carry the same row. React and plain JS.
- `layout.scrollbarStyle` (default `'thin'`, as before): `'native'` gives the browser's own scrollbars back (on Windows the classic wide bar with arrows). The thin styles stand aside instead of being overridden, because Chrome swaps the classic bar for the thin one when any scrollbar property or ::-webkit-scrollbar rule applies. React and plain JS; needs the stylesheet of this release.
- `layout.rowHeaderOverflow: 'unpin'`: the row headers are never pinned; the whole table scrolls sideways as one, each column at its width. In plain JS, which already scrolls the whole table, `'scroll'` and `'unpin'` do the same.

### Changed

- The corner button of the calculated-field window is named "Close" (it was "Cancel", like the button in its footer), in the React and plain JS windows.
- The Excel list and pivot files' columns are as wide as their heading and values (8 to 60 characters).

### Fixed

- Focus fell on `<body>` after Ctrl+Z on a big dataset: the cells' pane is swapped when the worker's result is painted, which can be long after the result arrives, and the 1.11.4 helper had stopped watching by then.
- On a phone (390-412 px wide, a 395-450 px pivot) the first data row sat below the visible part of the table, also with no filter fields.
- An export with no writer reported success, and so did an unknown export format.
- The subtotal and grand-total labels in Excel, PDF, print and clipboard ignored the app's `dictionary`: the file said 'Alt Toplam: X' where the screen said 'Toplam: X'.
- With many row fields on a small or zoomed screen the pinned row headers took the whole width and the data columns sat outside the box with no scrollbar to reach them.
- Under CSS `zoom` the field list window ran off the screen, and popups placed at pointer coordinates drifted.
- The filter area cap of B.61 followed the window's width, so a zoomed-in desktop (a 1280 px window with a ~640 px pivot) got no cap and 30 filter chips took the whole pivot.

## PivotGrid v1.11.4 — November 15, 2026

`@kanunilabs/pivotgrid-core@1.11.4` · `@kanunilabs/pivotgrid@1.6.4` · `@kanunilabs/pivotgrid-react@1.8.4` · `@kanunilabs/pivotgrid-enterprise@1.7.4` · `@kanunilabs/pivotgrid-react-enterprise@1.8.4`

_The arrow keys, Home, End, Page Up and Page Down move the selection with the focus, and Shift extends it. Also: the calculated-field window is a real dialog, the Move-to menu opens by touch, undo keeps focus in the pivot, getSummaryTable can return the shown value, and a phone keeps room for data below the filter strip._

### Before you upgrade

- A plain arrow key (or Home, End, Page Up, Page Down) now selects the cell it moves to, so `onSelectionChange`, `onSelectionStatsChange` and the Selection Detail panel follow the keyboard. Before, only the focus ring moved. An app that treated the selection as "what the user clicked" now sees it change on every key press.
- Ctrl or Cmd with an arrow, Home or End moves only the focus and leaves the selection; add Shift to extend it. Space selects the focused cell (and still sends `onCellClick`).
- On a screen 768 px wide or narrower the filter area takes at most 40% of the pivot's height and scrolls inside itself, also when `layout.filterAreaMaxRows` asks for more; a smaller cap and `filterAreaCollapsed` work as before. Before, a pivot of 400-500 px with many filter fields could be all filter area. The pivot needs a height of its own (an auto-height pivot is not capped).
- Adding or deleting a calculated field no longer writes a layout-undo step: `canUndoLayout()` stays as it was and `undoLayout()` does not spend a press on a step that changed nothing on screen. Placing the field in an area is still a step. Deleting a calculated field cannot be undone.
- `undoLayout()` and `redoLayout()` keep the focus: if it was in the pivot it stays in the pivot, and if it was on the button that asked and the step disabled that button, it moves to the pivot's grid element. An app that moved the focus itself after the call can drop that code; focus that was on `<body>` is left alone.
- The "Move to" and "Order" menu items open their submenu by touch, and a mouse click on a submenu that hover has just opened keeps it open (before, the click closed it). The plain JS grid also keeps only one of the two open.
- The calculated-field window closes on Escape and gives the focus back to the button that opened it; it is a dialog now (`role="dialog"`, `aria-modal`, named by its title). A page that listened for Escape itself while the window was open will see it close.

### Changed

- Arrow keys, Home, End, Page Up and Page Down move the selection together with the focus (one cell). With Shift they extend it from the anchor cell, shrink it when reversed and open it to the other side past the anchor. Ctrl or Cmd with these keys moves only the focus; Shift+Ctrl extends. Space selects the focused cell, Shift+Space extends, and Ctrl or Cmd+Space adds a rectangle in the Enterprise grids. Every change sends one `onSelectionChange` and one `onSelectionStatsChange`; React, Enterprise and the plain JS grid agree (the plain JS grid has no `onSelectionChange`).
- Leaving the data cells for the row headers by keyboard or mouse now clears the selection statistics (`onSelectionStatsChange(null)`); before, the last total stayed.

### Fixed

- After a mouse drag selection the focus is on the cell where the drag ended, so the next arrow key starts there instead of at the cell focused before the drag.
- The pane scrolls sideways to keep the focused column in view when a pivot has many columns (also with column virtualization, resized columns and right-to-left).
- Shift across the border between the row headers and the data cells no longer builds a rectangle that holds both.
- The calculated-field window (React and plain JS, Enterprise) is a dialog: it has a role and a label, Escape closes it, and focus returns to the button that opened it.
- The "Move to" and "Order" submenus open by touch. A tap used to open them through the mouse events a tap produces and the click that followed closed them again.
- Adding a calculated field no longer writes an empty step into the layout undo history.
- After `undoLayout()` and `redoLayout()` the focus is no longer dropped on the page: it stays in the pivot, or on the button that asked.
- On a phone the filter strip no longer fills the whole pivot: it stops at 40% of the pivot's height and the first data row is in the box.

### Added

- `getSummaryTable(controller, { display: true, locale })` (Enterprise): the value the grid shows, a percent of the grand total for example (97.6, not the raw 400), and a `formatted` text per data field and row. Without the option the result is as before: the raw aggregate.

## PivotGrid v1.11.3 — November 14, 2026

`@kanunilabs/pivotgrid-core@1.11.3` · `@kanunilabs/pivotgrid@1.6.3` · `@kanunilabs/pivotgrid-react@1.8.3` · `@kanunilabs/pivotgrid-enterprise@1.7.3` · `@kanunilabs/pivotgrid-react-enterprise@1.8.3`

_Data import is removed: the grid no longer reads files, and the xlsx dependency is gone._

### Before you upgrade

- Data import is removed. The footer import menu, file drag-and-drop, paste import, the import preview and status dialogs, and the controller methods importFile, importFiles, importFromUrl, importData and importText are gone, with the options onImportData, previewImport and toolbar.showImportFile and the state importStatus and importPreview. Remove them from your code; give the grid its rows through data, setData or applyTransaction.
- pivotgrid-core no longer depends on xlsx in any form. The Excel reader was the only user of it, and the xlsx releases on npm carry known security flaws (CVE-2023-30533, CVE-2024-22363). Exports are unchanged.
- A file dropped on the grid is no longer read. To also stop the browser opening it in the tab, prevent dragover and drop on your own container.
- `onFieldMove` is now asked on every move, not only a drag: the "Move to" menu, the field list (checkbox, "Select all", hide), the chip keys and `changeFieldArea`, `showFields`, `removeField` and `moveField`. A handler written for drags alone now sees these too; `e.cancel()` works on all of them.
- The field list checkbox puts a field in the area of its definition, else the first of its `allowedAreas`, and only then data for a number and rows otherwise. A hidden field defined in the filter area used to come back as a row or a measure.
- On a touch screen a finger drags a chip only after a hold (`touchDragDelay`, 200 ms); a finger that moves sooner scrolls the page. Before, a finger on the chip's name dragged after 8 px.
- A field with `visible: false` is now hidden everywhere: not in the field list, not in an area, not computed as a row, column or value, not exported. It used to show everywhere. A filter on it still applies, and its `area` is kept, so `visible: true` puts it back.
- On a touch screen a pivot chip no longer shows its filter and remove buttons: one "…" opens the chip menu, which now has "Filter field". In a row header cell there is no "…" either (hold the chip for its menu). With a mouse nothing changes.
- The footer's "Expand all" button now opens every level at once (`expandAll({ allLevels: true })`); it used to open one more level per click. On a large pivot that is every row and column member: the fully expanded result is computed first.
- Path keys of a member with an empty text at the first level changed (see Fixed): "|" for that member and "||x" below it, where it used to be "" and "x". Paths as arrays (`cellClick`, `getCell`, the selection) are unaffected. If you stored such keys by hand, rebuild them with `pathKey`.
- Undo (`undoLayout`) also puts back the open branches and summary sorts that the change being undone dropped.

### Changed

- File, clipboard and drag-and-drop import with its preview and column mapping, in Community and Enterprise, React and plain JS.
- The xlsx peer dependency.
- A measure chip names its summary type in the language of the grid ("Toplam", "Ortalama" with `locale="tr"`) instead of the raw `SUM`; a `custom` summary shows no badge. In Turkish, Min and Max now read "En küçük" and "En büyük" everywhere.
- `setFields` from the app now defines the fields: the captions, groups and visibility it passes are what saved states are compared with, and the areas are where the field list puts a field back. Before, only the first definition counted, so translated captions were saved as the reader's and an old language came back after a reload. The grid's own calls pass `setFields(fields, { definition: false })`.
- `expandAll({ allLevels: true })` opens everything with the `'*'` sentinel above 5,000 groups or when the full computation fails (reported through `onError`); then a single branch cannot be closed. Only groups are stored, not leaves.
- With a controlled `fields` prop, a split field group is reported once through `onFieldsChange` as joined.
- In a tabular export with `totalsPosition: 'top'`, the subtotal row keeps the group's name and reads "Total: <group>" in the next column.
- `clearLayoutHistory()` is also on the React ref and the plain JS grid; it and the undo flags now notify listeners, and a step updates the fields and the flags together.

### Added

- `getState().isComputing` and the `onComputingChange(computing)` option and prop: whether a change is still being computed, so `engineResult` is not yet current.
- `getState().noData`: the grid has no rows to show and none are queued.
- Every footer button, and the saved-view row actions, has an `aria-label` in the language of the grid.
- `allowedAreas` on a field: the areas it may sit in. Dragging shows "not allowed" over the others and refuses the drop, "Move to" disables them, and the field list and the API keep to them. `controller.canMoveFieldTo(id, area)` and `controller.getFieldHomeArea(id)` answer for your own controls.
- `touchDragDelay` (React and plain JS): how long a finger holds a chip before it drags. `touchChipTap` (React): what a tap on a chip does, `'sort'` (default), `'menu'` or `'none'`. Holding a chip still opens its menu, also on iPhone and iPad.
- While dragging a field, the field list, its boxes, the area strips and the pivot body scroll at their edges; the page around the grid does not.
- `fieldChooserMode="panel"` (React): the field list opens beside the pivot instead of over it. The pivot stays usable, and fields drag both ways between the list and the pivot. The width is `--pivot-chooser-panel-width`; at 768 px or narrower the full-screen list opens instead.
- `layout.filterAreaMaxRows`: the filter area grows to at most that many rows of chips, then scrolls inside itself (React also while a field is dragged at its edge). A button in the filter area collapses it to one row and shows how many of its fields filter; that is `layout.filterAreaCollapsed`, saved with the layout. Neither re-runs the pivot.
- `layout.showTotalsForSingleValues: false` leaves out the subtotal of an expanded group with a single child, as a row and as a column, on screen and in every export. `layout.showGrandTotalsForSingleValues: false` leaves out an axis's grand total when its top level has a single member. Both default to `true`.
- `expandAll({ allLevels: true })` on the controller and the React ref opens every row and column level in one call; each group is opened by its own path, so a branch can still be closed on its own. A plain `expandAll()` still opens one more level.
- A `loading` option (React prop, plain JS config, controller option) and `setLoading(loading)` (controller, plain JS grid): while the application is still fetching the rows, a grid with no result shows its loader instead of the "no data" text, so `data={[]}` filled by `applyTransactionAsync` no longer flashes the empty text before the first chunk. A result already shown stays; `getState().noData` is false while loading; `exportData` does not wait for it.
- `getPersistedState()` now keeps a field's `caption`, `group`, `groupIndex` and `visible` where they differ from the app's definition; `applyPersistedState`, `stateStoring` and `mergePersistedState` (layout part) put them back. A snapshot without them returns the field to its definition, so a caption the app translates later is not frozen by an old snapshot.
- `controller.setFieldRepair(repair)`: a function run inside every change of the fields, before it is stored and reported. The Enterprise grids pass `keepFieldGroups` through it.
- Layout undo and redo: `undoLayout()`, `redoLayout()`, `canUndoLayout()` and `canRedoLayout()` on the controller, the React ref and the plain JS grid, and `getState().canUndoLayout` / `canRedoLayout`. A step is a change of where fields sit, their order, whether they are shown, their caption or group; filters, sorting and summaries are not steps and undo leaves them. The last 20 steps are kept; `setFields` counts as a step, `applyPersistedState` and loading a report start over, `clearLayoutHistory()` empties it. No keyboard shortcut is bound.

### Fixed

- With no rows (`data={[]}`, or an empty `applyTransactionAsync` batch) the React grid showed "Loading..." forever. Both renderers now show the `emptyData` text, which the dictionary can change.
- An export asked for while a filter or field change was still computing could write the previous result. `exportData` now waits for the change, in every format, with no time limit; it rejects with an `AbortError` if the grid is destroyed while it waits, which the footer and the React ref ignore.
- A filter changed in the same moment as the field layout (moving a field, then filtering) dropped the layout change.
- A dragged field landed where the drag overlay touched, not where the pointer was: 3 px of a chip in another area took the drop. The target is now what is under the pointer, and the insertion mark and the drop always agree (the mark could show the previous target).
- Dragging from the field list onto a chip or the empty part of an area box went nowhere or to another box, and the pivot under the list still took drops. Dropping a field on the "All Fields" list did not hide it.
- With fields in the row area, the empty part of the row header cells was no drop target; it is now the end of the row area.
- Dragging from the field list showed “Field” instead of the field's name, and the source chip travelled with the drag instead of staying in place.
- Drag announcements for screen readers read internal ids in English; they now say the field's caption and the area in the language of the grid.
- The field list's drag handle is a 44 px target on touch screens.
- On a phone the field list and its four boxes could not be scrolled: only the first screen of "All Fields" was reachable. The list now scrolls by finger, down to the Values box.
- On a touch screen the chips in a narrow row header showed no name, their invisible buttons took the taps (a tap opened the filter, the right edge removed the field), and on a phone they were narrower than 44 px.
- On a phone the chip menu could open below the bottom of the screen. Every pivot menu and the header filter now open inside the visible part of the window, upward when there is no room below.
- The Field Settings dialog was wider than a 360–390 px phone screen.
- Another element of the page going full screen (a video, a chart) turned the React pivot into its full-screen look on top of it.
- The floating field list no longer jumps to the centre right after it opens.
- Subtotal rows in every export (Excel, PDF, CSV, print, clipboard) carried the numbers of the level above them — for a top-level group, the grand total — and in the tabular layout had no label. They now carry their group's numbers and read "Total: <group>" as on screen.
- With field groups (Enterprise), moving one member made `onFieldsChange` fire twice, first with the group split, then joined. The group is now joined inside the same change, so it fires once.
- A field group could be pulled into an area one of its members' `allowedAreas` forbids. The move is now refused for the whole group: `canMoveFieldTo` says no for every member, menus and drag targets follow it, and a `setFields` that splits the group there is joined back where the other members are.
- A click in the middle of a field chip in a row header cell opened its filter or removed the field instead of sorting it: the chip fills the cell, and its filter and remove buttons, invisible until hovered, sat right after the caption. They now sit at the chip's end, and the rest of the chip sorts (both renderers).
- A drop in the middle of a wide chip landed on the other side from the insertion mark (React): the last intent was worked out after the mark had been reset.
- A field drop waiting on an asynchronous `onFieldMove` overwrote any change made meanwhile; it now applies to the fields as they are when the answer comes. A rejected `onFieldMove` goes to `onError` and the drag always ends; it used to leave an unhandled rejection in plain JS.
- Each plain JS field menu left a window listener behind until the next resize.
- "Move to" enabled an area another member of the field group may not go to, and clicking it did nothing; both renderers now ask `canMoveFieldTo`.
- On a touch screen, a long press whose touch the browser cancelled ate the next tap.
- Plain JS: `fieldChooserMode` now warns that it is React only.
- A member whose text is empty at the first level of rows or columns had the pivot's own key, and the members under it lost a part: its children were keyed like another member's ("12.5" for ["", "12.5"], the key of ["12.5"]). Opening one branch could open the other, and a sort by that column used the grand total. Such a path now has its own key (a leading separator: "|", "||x"); every other key is unchanged. `pathKey(parts)` and `pathParts(key)` build and read keys; use them instead of `join('|')` and `split('|')`. This is also why a check on a saved state could not count a key's levels: with the old keys an empty first member made a path look one member shorter. Keys now have one part per level. The scheme assumes member values do not start with or contain "|"; such a value can still share a key with another member, as before.
- Open branches (`expandedPaths`, `expandedColPaths`) and summary sorts were kept after a field change made them meaningless: with rows [Code, BuyPrice] and "A|12.5" open, moving BuyPrice to the filter area left the path and the sort in the grid and in `getPersistedState()`. A path now stays while the leading fields it was built from are unchanged and goes otherwise, in the same update, for a drag, the menu, the field list, hiding or deleting, `setFields`, a controlled `fields`, undo and redo; a summary sort goes with the fields it names. A state loaded from an older version loses the paths with more members than their axis has fields. `'*'` is never dropped.
- React: a field dragged to its target in one move and released there (no settling move) moved nothing, in Chrome, Firefox and WebKit. The drop read the pointer and the droppables from a collision run that dnd-kit had not made yet; it now reads the browser's own pointer events and the droppables as they are. The plain JS grid was not affected.

## DataGrid v1.10.16 — November 15, 2026

`@kanunilabs/datagrid-core@1.10.16` · `@kanunilabs/datagrid@1.9.16` · `@kanunilabs/datagrid-react@1.8.16` · `@kanunilabs/datagrid-enterprise@1.10.16` · `@kanunilabs/datagrid-react-enterprise@1.9.16`

_The browser's own scrollbars on request (`scrollbarStyle: 'native'`)._

### Added

- `scrollbarStyle` (default `'thin'`, as before): `'native'` puts `kanuni-datagrid-native-scrollbar` on the grid and the themed thin scrollbar styles stand aside, so the browser draws its own (on Windows the classic wide bar with arrows). An unthemed grid already used the browser's. React, plain JS and both Enterprise wrappers; the stylesheet of this release is needed.

## DataGrid v1.10.15 — November 14, 2026

`@kanunilabs/datagrid-core@1.10.15` · `@kanunilabs/datagrid@1.9.15` · `@kanunilabs/datagrid-react@1.8.15` · `@kanunilabs/datagrid-enterprise@1.10.15` · `@kanunilabs/datagrid-react-enterprise@1.9.15`

_The import wizard and the CSV import helpers are removed; the grid no longer reads files._

### Before you upgrade

- The Enterprise import wizard is removed: openImportWizard, EnterpriseGridApi.openImportWizard, the onImport option and GridImportWizard in React. The helpers importCsv, parseCsv, sniffDelimiter, autoMapColumns, buildImportPlan and applyImportMapping are removed from datagrid-core and datagrid-react. Clipboard copy and paste into cells, fill and editing are unchanged.

### Changed

- The import wizard (file, column mapping, commit) and the CSV import helpers.

## PivotGrid v1.11.2 — November 14, 2026

`@kanunilabs/pivotgrid-core@1.11.2` · `@kanunilabs/pivotgrid@1.6.2` · `@kanunilabs/pivotgrid-react@1.8.2` · `@kanunilabs/pivotgrid-enterprise@1.7.2` · `@kanunilabs/pivotgrid-react-enterprise@1.8.2`

_Ratios that stay blank on a division by zero, helper measures, touch-sized targets, translated Excel totals, the summary as a flat table, several selected cells with their source rows, field groups, and saved layouts kept apart from their filters._

### Before you upgrade

- A summary formula or function that has no number — a division by zero, null, NaN, Infinity or a throw — now leaves the cell blank instead of 0, on screen, in the selection totals and in exports. The function compileSafeFormula returns gives NaN on a division by zero; a row-level formula still reads it as 0.
- On a touch screen the field chips are wider (44px controls) and the column header filter funnel sits at the end of the cell and is always shown.
- On EnterprisePivotGrid, Ctrl/Cmd+click now adds a rectangle to the selection and Shift+click extends it, instead of starting a new one.
- The React Enterprise Excel files carry the grid language for the grand-total and subtotal labels and the sheet names.

### Added

- `helper: true` on a data field: computed and read by summary formulas, never drawn or exported.
- `density: 'touch'` (45px rows and headers) and 44px touch targets on the footer, menus, chip controls and column header controls.
- `onLargeExport({ format, size, limit, unit })`, asked before a styled Excel export gives way to the plain .xls one; `false` cancels.
- Enterprise: `getSummaryTable(controller)` and `exportSummaryTable(controller, options)`, the summary as a flat table regardless of what is expanded.
- Enterprise (React): several selected cells — Shift, Ctrl/Cmd, a touch long press — with `onSelectionChange`, `ref.getSelectedCells()` and `ref.getRecordsForSelection()` (each source row once).
- Enterprise: field groups (`group`, `groupIndex`) that move as one.
- `splitPersistedState` / `mergePersistedState` to keep a layout apart from its filters; `clearAllFilters()`; `openFieldChooser()` / `closeFieldChooser()`; `controller.getCellRecords(rowPath, colPath)` and `controller.computeSnapshot()`.

### Fixed

- A division by zero in a ratio showed 0, which read as a real value.
- On a touch screen the column header filter funnel was a 10px glyph shown only on hover, and the chip controls were 24px.

## DataGrid v1.10.14 — November 13, 2026

`@kanunilabs/datagrid-core@1.10.14` · `@kanunilabs/datagrid@1.9.14` · `@kanunilabs/datagrid-react@1.8.14` · `@kanunilabs/datagrid-enterprise@1.10.14` · `@kanunilabs/datagrid-react-enterprise@1.9.14`

_Multi-select on cards, a typed-open lookup that never keeps its opening letter, the whole focused-row event in React, a read-only look at the open editor, and reasons for disabled check-box items._

### Before you upgrade

- In card view with `selection.checkboxColumn`, a press on a card now only focuses it; each card has its own check box for selecting. Before, the press also toggled a selection the card did not show.
- A lookup opened by typing a letter goes back to the cell's value on the Escape that closes its list, and on any leave that picked nothing (unless `allowCustom` or `clearable` with emptied text).
- The React `onFocusedRowChanged` prop now receives the whole event (`rowKey`, `previousRowKey`, `row`, `reason`).

### Added

- Card view selection (React): a check box on every card, 44 px on a touch screen; select-all and the selected count in the card bar; a selected card styled apart from the focused one.
- `controller.getEditSession()` and `controller.isEditorDirty()`: what is open for editing and whether it changed, read-only.
- `lookup.onDisabledItem({ item, row, rowKey, columnId })` for a press on a disabled `lookup.multiple` item, and `disabled: { reason, visible: false }` to give the reason only as a tooltip and to screen readers.
- `aria-controls` and `aria-activedescendant` on the `lookup.multiple` editor.

### Fixed

- A lookup opened by typing could keep the opening letter after Escape closed the list and Tab moved on.
- A page rule `* { box-sizing: border-box }` gave lookup lists a 2 px scroll bar.

## DataGrid v1.10.13 — November 12, 2026

`@kanunilabs/datagrid-core@1.10.13` · `@kanunilabs/datagrid@1.9.13` · `@kanunilabs/datagrid-react@1.8.13` · `@kanunilabs/datagrid-enterprise@1.10.13` · `@kanunilabs/datagrid-react-enterprise@1.9.13`

_Lookups that finish the same way whoever ends the edit, code boxes that complete the code, a form lookup with loading, accessibility and dark mode, check-box sets read from each row with 64-bit values, printing every column, and a row-leaving check that sees the finished cell._

### Before you upgrade

- A lookup editor now opens with its list CLOSED; focusing or clicking the box no longer opens it. It opens on typing, the drop button, F4, Alt+Down, Space with `editKeys.space`, or at once with `openOnEdit`. On a touch screen, where F4 and Alt+Down are missing, consider `lookup.dropButton: true`. `LookupBox` has `openOnFocus` for the old behaviour.
- An item reached with the arrow keys is now written by Tab and by leaving the cell, not only by Enter. A list that was only opened still writes nothing.
- Escape that closes a lookup list now also drops what was typed.
- In a code + name lookup (`linkedColumns`), `typeAhead` now completes the code, which is what the box shows. Set `lookup.display: 'name'` to show and complete the name.
- `stopEditing()`, `ctx.commit()` and every other commit now run the open editor's own leaving rule first: the completed or highlighted lookup item, a cleared date. An editor opened by typing a letter that picks nothing goes back to the cell's value instead of storing that letter.
- With `onRowLeaving`, the open cell is committed before the check is asked; if the check then refuses, focus stays on the row and the value stays written.
- `matchFold: 'tr-loose'` and `'case-accent'` (and `searchFold`) now also fold é, ä, ñ, ł and the other Latin letters with marks.
- `isRowLeaveBlocked(rowKey)` is now true only for a row a move was refused to during the same gesture, not whenever focus is on another row.

### Added

- `lookup.display` (`'code' | 'name' | 'both'`) and an item's own `code`: the box shows and completes that text, `value` can be the record's identity, and `linkedColumns.code` receives the code.
- `matchMode` `'none'` for one field of a code + name list, for a name box that does not find codes.
- `lookup.hoverHighlights` (default on) and `lookup.openHighlight` (`'none' | 'current'`): whether the pointer moves the highlight, and whether the list opens on the current item or the first.
- `LookupBox`: `items={null}` or `loading` (typing is kept and checked once the list arrives, with a "Searching…" line in twelve languages), `error`, `dark`, emptying clears the value, keys it does not use reach the form, `id`, `name`, `required` and `describedBy` on the box, a "…" button that can be a Tab stop, button order and names, `--kanuni-lookup-box-height`.
- Lookup accessibility: `aria-controls`, `aria-activedescendant` with option ids, `aria-autocomplete="both"` with `typeAhead`, `aria-keyshortcuts`; 44 px buttons on a touch screen.
- `lookup.multiple`: a function `items` gives each row its own list and `disabled` reasons; ticking changes only that item's bit; `multiple.emptyText`; bit sets as signed 64-bit numbers, bigints or decimal strings, written back in the same type; 44 px rows on a touch screen. New helper `toggleMultipleValue`.
- `focusedRowChanged.reason` (`'move' | 'view'`), the `rowLeaveRefused` event, and `focusRow` answering whether focus moved.

### Fixed

- Printing drew only the columns in view; every column is printed now.
- A cell being edited lost its editor when the grid scrolled sideways past it.
- A press on another row asked `onRowLeaving` before the open cell was finished, so the check did not see what was typed.
- A touch long press never got the row menu while `onRowLeaving` was set.
- Escape in a modal dialog that keeps the key to itself no longer leaves a lookup list open.
- A function lookup list still loading showed a disabled box that dropped what was typed.

## DataGrid v1.10.12 — November 11, 2026

`@kanunilabs/datagrid-core@1.10.12` · `@kanunilabs/datagrid@1.9.12` · `@kanunilabs/datagrid-react@1.8.12` · `@kanunilabs/datagrid-enterprise@1.10.12` · `@kanunilabs/datagrid-react-enterprise@1.9.12`

_Lookups that open your own picker, show their list as a table, resize and work as a form field; a check-box set in one cell; a hook that can keep focus on a row; accent-insensitive quick search; totals kept on the bottom edge; and keyboard moves that keep the row clear of the header and totals._

### Before you upgrade

- Enter inside a custom editor (`renderEditor`) now commits it the way it commits the built-in editor: down, or stay with `editKeys.enter`; Shift+Enter stays. Enter on a button, link, select, text area or an open combo box inside the editor is still that element's. An editor that uses Enter itself calls `event.preventDefault()` in its handler, as it already does for Escape and Tab.
- F4 in a lookup editor now opens and closes its list; before, it went on to the page. Alt+Down opens it and Alt+Up closes it.
- The plain JavaScript grid now focuses the clicked cell before it selects the row, as the React grid does, so `focusChanged` comes before `selectionChanged` on a click.

### Added

- `onRowLeaving(from, to)`: asked before focus leaves a row by any road (a click, the arrow keys, Tab, Enter, `focusCell`, `focusRow`, `moveFocus`, `clearFocus`). Return `false` to keep focus there, and the click selects nothing either; return a Promise to decide later, after a server check or a dialog. `focusCell` now answers whether focus moved.
- `searchFold` (default `'case'`, as before): quick search and find fold text in the lookup editors' `matchFold` modes; `'tr-loose'` makes "siparis" find "Sipariş" and marks the match. `controller.setSearchFold` changes it on a live grid.
- `pinFooterToBottom` (default off): the totals row stays on the grid's bottom edge when the rows end higher.
- `lookup.onPickerOpen` (React): a "…" button in the open editor, and Ctrl+Enter, open your own picker. The item it resolves with is written like a pick from the list, `linkedColumns` included; while it is open, focus leaving the grid neither commits the cell nor sends the row.
- `lookup.listColumns` and `lookup.dropButton` (React): the list as a table with a header row, and a ▾ button that opens it.
- `lookup.listResizable` (React): a grip on the open list's corner to make it wider and taller, kept per column until the page reloads.
- `LookupBox` (React Enterprise): the cell's lookup editor as a form field, with the same list, matching and keys.
- `lookup.multiple` (React editor): several items in one cell, as an array or a bit set (`{ bits: true }`). The cell shows the picked items' `short` labels joined with `;`; the editor is a list of check boxes, and an item with `disabled` cannot change.
- `numberFormat.restrictInput` (React, default off): typing that can never become a number in the column's locale is refused as it is typed; a paste is taken when the whole text reads as a number.
- `rawValue` in the `renderEditor` context: the column's stored field, before `valueGetter`.

### Fixed

- A row behind the totals row was taken for visible, so a row added with ArrowDown at the end stayed under the totals. Keyboard moves, `scrollToRow` and `scrollToCell` now keep the row between the header and the totals row.
- With `lookup.typeAhead` and `lookup.openOnEdit`, the letter that opened the editor was not completed.
- Best fit (all columns) left the widths of columns scrolled out of view unchanged.
- The grids threw where there is no `CSS` global, as in some test environments.

## DataGrid v1.10.11 — November 5, 2026

`@kanunilabs/datagrid-core@1.10.11` · `@kanunilabs/datagrid@1.9.11` · `@kanunilabs/datagrid-react@1.8.11` · `@kanunilabs/datagrid-enterprise@1.10.11` · `@kanunilabs/datagrid-react-enterprise@1.9.11`

_Lookup editors that complete the typed text in the box and pick the closest item with Tab or Enter, like a desktop lookup combo._

### Added

- `lookup.typeAhead` (default off): when the label of an item starts with the typed text, the box shows that label with the added part selected and the item is highlighted. Enter, Tab and leaving the cell write the highlighted item; Tab then moves on. Text that matches nothing writes nothing. With `matchMode: 'contains'` and no label starting with the text, nothing is completed and the first match is highlighted. Backspace removes the completed part and does not complete again, and an input method word is completed once it is finished. Ignored with `allowCustom`.

## DataGrid v1.10.10 — November 5, 2026

`@kanunilabs/datagrid-core@1.10.10` · `@kanunilabs/datagrid@1.9.10` · `@kanunilabs/datagrid-react@1.8.10` · `@kanunilabs/datagrid-enterprise@1.10.10` · `@kanunilabs/datagrid-react-enterprise@1.9.10`

_Lookup editors that open their list at once, reject text with no match while typing and ignore accents, drag selection that scrolls the grid at its edge, and printed pages that repeat the column headers._

### Before you upgrade

- Dragging a range selection with the mouse now scrolls the grid on its own when the pointer reaches or leaves the scroll area. There is no option: it is the default behaviour. Pinned columns stay in place.
- Printing from the browser (`window.print()`) with `printLayout` now repeats the column headers on every page. Set `printLayout.repeatHeader: false` to get the old single header. During printing the grid is briefly swapped for an equivalent table, and it is put back afterwards.
- `exportToPdf` no longer splits a row across two pages.

### Added

- `lookup.openOnEdit` (default off): the editor starts with its list open, however it was started. When typing started it, the list is already filtered by that character and the caret sits after it.
- `editKeys.space` (default off): Space on a focused lookup cell opens its editor.
- `lookup.restrictInput` (default off): text that matches no item (by `matchMode`) is refused as it is typed, pasted or dropped. Deleting is always allowed and input method composition is checked once it ends. Ignored with `allowCustom`.
- `lookup.matchFold`: `'none' | 'case' | 'case-accent' | 'tr-loose'`. The default is `'case'`, as before; `'case-accent'` also ignores accents ("cay" finds "Çay") and `'tr-loose'` also treats ı and i as one letter.
- `printLayout.repeatHeader` (default on): column headers, grouped ones included, repeat on every printed page.

### Fixed

- Selecting a range with the mouse did not scroll a grid wider or taller than the screen.
- Printed pages could cut a row in two.

## DataGrid v1.10.9 — October 31, 2026

`@kanunilabs/datagrid-core@1.10.9` · `@kanunilabs/datagrid@1.9.9` · `@kanunilabs/datagrid-react@1.8.9` · `@kanunilabs/datagrid-enterprise@1.10.9` · `@kanunilabs/datagrid-react-enterprise@1.9.9`

_In-cell editing for document forms in React: the app reaches the editing engine, rows read new and removed, a locale-aware number editor, code + name lookups, a custom-editor contract for picker dialogs, a configurable keyboard with a key hook, one save per row in cell mode, row class names and tappable cards._

### Before you upgrade

- With `rowStatus` on, an added row now reads `new` and a row marked for removal reads `removed` (both used to read `changed`). They have their own colours, `--dg-row-status-new` and `--dg-row-status-removed`, and with `icons` a plus and a cross.
- After a successful `onSave`, removed rows leave the grid's data, as they already did with `crud`. A row your `onSave` already took out of your own state, or a new `dataSource`, is left alone.
- A successful save takes what it sent out of the undo history: in `batch` everything up to that save, in `cell` the saved change. Ctrl+Z no longer brings back a change the server already has.
- The built-in editors no longer stop F1-F12 and the Ctrl / Alt combinations they do not use, so page shortcuts see them while a cell is being edited. The grid ignores a key whose event is already `defaultPrevented` and any combination with Alt (AltGr typing still opens the editor): Alt+PageDown no longer scrolls the grid.
- A `renderEditor` editor: Escape now cancels it and Tab on its last focus stop commits and moves to the next cell, unless the editor calls `preventDefault()` on the key. Tab used to leave the grid, Escape did nothing.
- A lookup column with `allowCustom` is edited with the grid's own dropdown instead of the browser's `<datalist>`. A lookup whose list is empty also gets the dropdown (it used to fall back to a text box that accepted any value).
- On touch screens (`pointer: coarse`) every cell and form editor uses at least a 16 px font, and lookup items are 44 px tall, so iOS does not zoom in on focus.
- Tapping or clicking a card in `cardView` now focuses its row, so `onFocusedRowChanged` fires for cards. Cards are at least 44 px tall.
- Turkish: the "no match" text of the lookup editor (`editNoMatch`) reads "Kayıt bulunamadı". The other languages are unchanged.
- On a page without a secure context (plain HTTP) Ctrl+V now pastes, through the browser's own paste event. It did nothing there before; secure pages are unchanged.
- Two row-status colours were raised to a 3:1 contrast against the row: `saving` in the kanuni theme and `error` in the dark nord scheme.

### Added

- `editing.onReady(api)` and `controller.editing`: the `EditingApi` — `hasChanges`, `getChanges`, `saveChanges`, `cancelChanges`, `addRow`, `removeRow`, `restoreRow`, `isRemoved`, `getRowStatus`, `getRowStatusError`, `getErrorRowKeys`, `effectiveRow`, `undo` / `redo`, `isDirty`, `revertRow`, `revertCell`, `setRowValues`.
- `SaveRejected`: throw `new SaveRejected([{ rowKey, columnId?, message }])` from `onSave` to mark the rows the server refused; `SaveResult.rows` lists them and nothing is counted as saved.
- The `changesChanged` event (`{ dirty, count }`), `editing.batchBar: false` and `editing.history: false`.
- With `rowKey` set to a field, a row added by `addRow`, Insert, arrow-down or Tab keeps the key the app gives it in `onInitNewRow`; the grid no longer writes its own temporary key over it.
- `rowStatus`: the `new` and `removed` statuses and `rowStatus.strike`.
- `numberFormat` on a column: `locale`, `decimals`, `min`, `max`, `step`, `allowNegative`, `thousands`. The editor reads "1.234,5" in Turkish, refuses "1.5" instead of guessing, rounds to `decimals`, opens the decimal keyboard on phones and steps with the arrow keys.
- Typing `-` or the decimal separator on a focused number cell, or a digit on a date cell, opens its editor.
- `renderEditor` context: `commit({ move })`, `suspendBlurCommit(on)`, `commitOnBlur`, `api.setRowValues(values)`, `rowKey`, `ariaLabel`, `invalid`, `errorId` — for a "…" button that opens a picker dialog and writes several fields in one step.
- Lookup: `linkedColumns: { code, name, id }`, `matchMode: { code, name }`, `map(item)`, `autoCompleteSingle` and `onNoMatch`. A function `items` list now shows labels in the closed cell; a dependent list clears a value that is no longer in it.
- Keyboard: `editing.editKeys` (`f2`, `enter`, `typing`, `doubleClick`), `insertKey: 'above' | 'below'`, `appendOnTab`, `deleteKey` (Ctrl+Delete, asks `confirmDelete`), `fillKeys` and `rangePaste`, and the grid-level `onKeyDown(event, ctx)` hook.
- `editing.commitOn: 'row'`: in cell mode a row is saved once, when the user leaves it; a new row nothing was written into is dropped instead of sent.
- `rowClassName({ row, node, rowKey, status, index })` on both React grids.
- `cardView.onCardPress`, `renderCard`, `height` and `edit: 'popup'` (React).

### Fixed

- Arrow-down on the last row added a row and then moved the focus to the first row.
- A double tap did not reliably open an editor on touch screens; the grid now counts the second tap itself.
- A card showed the stored value instead of a pending `batch` value, and a function lookup's code instead of its label.
- Lookup items overlapped by 8 px because of their padding.

## DataGrid v1.10.8 — October 29, 2026

`@kanunilabs/datagrid-core@1.10.8` · `@kanunilabs/datagrid@1.9.8` · `@kanunilabs/datagrid-react@1.8.8` · `@kanunilabs/datagrid-enterprise@1.10.8` · `@kanunilabs/datagrid-react-enterprise@1.9.8`

_In a short box the grid keeps its rows before the docked chart keeps its smallest size, a plain Excel file carries the column formats, and the header filter's value list is one Tab stop._

### Before you upgrade

- A plain Excel export (`styled: false`) now writes each column's number format — its `excelFormat`, `yyyy-mm-dd` for a date column, a thousands separator for a number column. Dates used to come out in Excel's own mm-dd-yy. Alignment, banded rows, the outline and the totals row still need `styled`.
- In a box too short for the chart toolbar as it wraps, the smallest chart and 80 px of rows, the docked panel carries `data-dock-compact`: its toolbar keeps to one line that scrolls sideways, and the chart may be shown below 120 px. The height kept in the chart state is unchanged.
- In the header filter only the active value is in the Tab order (the list itself while that value is scrolled away); the arrow keys, Home / End and Space work as before.

### Added

- `chartDockLayout` in the core: how the docked chart is shown in the box it shares with the grid, compact or not.

### Fixed

- The docked chart was fitted before its toolbar wrapped and only again when the box resized, leaving the grid 20-50 px of rows; the panel's own size is watched too.
- In a short box the chart's 120 px minimum and a wrapped toolbar left the grid fewer than 80 px of rows, or none.
- A plain Excel file ignored `excelFormat`.
- Every value in the header filter was a Tab stop of its own: a list of 30 took 30 presses of Tab to reach Apply.
- End in a long header filter list could not reach the last value (React), and dropped the focus on the page (plain JavaScript).

## DataGrid v1.10.7 — October 29, 2026

`@kanunilabs/datagrid-core@1.10.7` · `@kanunilabs/datagrid@1.9.7` · `@kanunilabs/datagrid-react@1.8.7` · `@kanunilabs/datagrid-enterprise@1.10.7` · `@kanunilabs/datagrid-react-enterprise@1.9.7`

_A hidden tab keeps its chart in the stored view, the docked chart fits a short box, and the toolbar Chart button can be left out._

### Before you upgrade

- A view-state part whose owner unregisters (`registerViewStatePart`) now stays in `getViewState().parts` with its last snapshot until the owner registers again, instead of leaving the view. A chart kept this way is handed back to the chart when it returns.
- `stateStoring` no longer writes when the grid is torn down unless a save is still waiting.
- The docked chart is shown no taller than leaves the grid 80 px of rows, on open and when its box resizes; the height kept in the chart state is unchanged.
- In Excel, numeric text with decimals ("12.50") in a column on the default number format keeps its decimals (`#,##0.00`).

### Added

- `toolbar.chart: false` leaves out the Chart button the Enterprise grids add to the toolbar; the chart still opens from code and the menus.

### Fixed

- A tab hidden with React's `<Activity>` lost its chart from the stored view: the chart unregistered as the tab hid, and `stateStoring` then wrote the view without it — every time, because its save timer was never cleared after it ran.
- The chart divider counted the grid's sticky column header, which sits inside the scroller, against its 80 px: in a 614 px dialog the grid kept one and a half rows. It counts rows only now.
- The docked chart opened at its full height in a short dialog and left the grid no rows until the divider was touched.
- The search box's error message was drawn at 11 px; it uses the grid's body size.
- Excel showed "12.50" from a database DECIMAL as 12.5.

## DataGrid v1.10.6 — October 28, 2026

`@kanunilabs/datagrid-core@1.10.6` · `@kanunilabs/datagrid@1.9.6` · `@kanunilabs/datagrid-react@1.8.6` · `@kanunilabs/datagrid-enterprise@1.10.6` · `@kanunilabs/datagrid-react-enterprise@1.9.6`

_The toolbar search box follows the grid, Excel reads values the way databases send them, and F3 works in the find box._

### Before you upgrade

- The toolbar search sends its text trimmed: `getSearchText()`, `filterChanged`, the saved view and a server source's `searchText` no longer carry the spaces typed around it.
- In Excel, text that reads as a number in a `number` column ("12.50", as MySQL sends a DECIMAL) is written as a number, and a date that names no day ("0000-00-00") as an empty cell, in the plain, styled and streamed exports. A whole number on the default number format is shown without a trailing decimal separator.
- The chart divider counts only the grid's rows (80 px) against its minimum, not the header and footer, so the chart can be dragged taller in a short box than in 1.10.5.

### Added

- `toolbar.search.ariaLabel` gives the search box its own accessible name.

### Fixed

- The toolbar search box read the grid's search only when it mounted (and never in plain JavaScript), so a view restored afterwards — a tab switched back to, a reload, a dialog opened with a search — filtered the rows under an empty box. It now shows the grid's search wherever it was set.
- The search box's debounce also ran when it mounted and could write its empty text over a search a saved view had just restored; it runs only after the user types.
- On a phone the search box takes the toolbar's whole first line, and on a touch screen it is a 44 px target with 16 px text, so iOS no longer zooms in when it takes focus.
- Excel showed a whole number on the default format as "1,234." and a zero date as "####"; a number column of numeric text could not be summed.
- F3 and Shift+F3 in the find box opened the browser's find instead of stepping through the matches.

## DataGrid v1.10.5 — October 28, 2026

`@kanunilabs/datagrid-core@1.10.5` · `@kanunilabs/datagrid@1.9.5` · `@kanunilabs/datagrid-react@1.8.5` · `@kanunilabs/datagrid-enterprise@1.10.5` · `@kanunilabs/datagrid-react-enterprise@1.9.5`

_Streamed CSV and Excel exports with no row cap, a divider to resize the docked chart, and keyboard and focus fixes for the filter dialogs inside a modal._

### Before you upgrade

- The Turkish name of the missing-value row in the header filter (and of a blank group) is now "(Boş Olanlar)", matching "(Boş Olmayanlar)". A test that looks for "(Boşlar)" must look for the new word.
- The filter panel sentence names an empty string "(empty text)" and a value of spaces "(1 space)", "(2 spaces)", as the header filter lists them; a missing value reads "(Blanks)". `FilterLabels` gains `emptyText`, `space` and `spaces`: a full `FilterLabels` object of your own needs them.
- Under `portalContainer` the layer the grid puts its popups in now covers the viewport (it was zero-size) and passes clicks through to the page; what is put in it takes them. An app that styled `.kanuni-datagrid-portal-host` should check its rule.
- The grid's error message (a failed menu action) is marked `data-kanuni-portal="toast"` instead of `"popup"`.
- With the Enterprise side bar, card view compares its breakpoint with the grid and the side bar together, so a grid with a side bar switches to cards at the same screen width as one without.

### Added

- `stream: true` on `exportToCsv` and `exportToExcel` writes the file as the rows arrive instead of building it in memory: the view in slices, a server source's pages, or its own new `DataSource.exportRows(request, signal)` stream. With `remote.exportMaxRows: Infinity` there is no cap. Excel is written by a writer of our own (no library to load): typed values, each column's number format, widths, a frozen header, a filter, and a new sheet past Excel's row limit. Cancel or a failure hands over no file; `writable` takes the bytes straight to a file the user picked and is aborted, never closed, when the export does not finish. 1.2 million rows took 7 seconds and opened in Excel.
- A divider between the docked chart panel and the grid: drag it (or use the arrow keys on it) to make the chart shorter and give the grid its rows back, which matters most in a modal. The grid keeps at least 120 px of a box of fixed height. The height is the chart state's new `dockHeight`, kept by `getViewState()`.
- `printLayout.onLimit({ rows, maxRows })` is called when a print is asked for over `maxRows`, so the app can say that only the rows on screen were printed.
- `data-kanuni-portal="toast"` marks a short-lived message, apart from `popup` and `tooltip`.

### Fixed

- The filter builder took no focus when it opened (React), Tab walked out of it into the page behind, and closing it left focus nowhere. It now takes focus, closes on Escape, keeps Tab inside, and gives focus back to the column header or the button it was opened from.
- The header filter's "(Custom…)" entry could be reached with Tab but not opened: Enter on it applied the filter, and Space on a button ticked a value. A button in the popup now takes its own Enter and Space.
- Inside a modal centred with a CSS transform (`portalContainer`), the filter builder, the calculated column dialog and the other full-screen dialogs opened off the screen to the left and slid away when the window was resized. They open centred and stay there.
- In a modal, Escape on the header filter sent focus to the modal instead of back to the column; it goes to the column header.
- The calculated column dialog showed a fixed English example, "[Price] * [Qty]"; it shows one built from the grid's own columns.

## DataGrid v1.10.4 — October 24, 2026

`@kanunilabs/datagrid-core@1.10.4` · `@kanunilabs/datagrid@1.9.4` · `@kanunilabs/datagrid-react@1.8.4` · `@kanunilabs/datagrid-enterprise@1.10.4` · `@kanunilabs/datagrid-react-enterprise@1.9.4`

_The header filter tells blanks, empty text and spaces apart, opens the filter builder from a column, and keeps Tab and focus inside._

### Before you upgrade

- On a text column, `=` and `in` compare as text: an empty string, a space, `0` and `012` are four different values, where they used to be read as numbers and match each other (`012` equalled `12`). A number or bigint column compares numerically as before.
- The header filter lists a missing value (null) as its own row, and an empty string and each run of spaces as rows of their own, where all of them were one Blanks row. A filter you stored with the old behaviour keeps working; a list you build from the checklist now names the value it means.
- The header filter popup is named from the dictionary (`filterPopupAria`, "{column} filter" in English). A test that finds it by an English aria-label in another language must use the grid language.
- The filter builder dialog, its bar and the Import button read their text from the grid dictionary. In English the words are the same; in any other language they now follow the grid.
- When your app passes `onOpenFilterBuilder` to an Enterprise grid it is now the one called by the filter panel and the header filter, with `{ columnId }`; before, the grid replaced it with its own. `controller.openFilterBuilder()` opens the grid's own builder whichever you pass.

### Added

- The header filter lists a null as a Blanks row, an empty string as an empty text row, and a run of spaces as a row naming how many (`filterEmptyText`, `filterSpace`, `filterSpaces`, in 12 languages). When the column has blanks, a Non-blanks row selects everything else and applies as one `isnotnull`; a stored `isnotnull` reopens with that row ticked. Server-side lists show the same rows.
- A custom entry at the top of the header filter opens the filter builder on that column, with the column's condition from the filter or an empty one to fill in. It shows wherever a builder can be opened: the Enterprise grids, or an app that passes `onOpenFilterBuilder` (which now receives `{ columnId }`).
- `filterBuilderBar: false` leaves out the bar above the grid that `filterBuilder` adds (Enterprise, both renderers); the builder is still opened from the filter panel, the header filter and code.
- `controller.openFilterBuilder(columnId?)` asks an Enterprise grid for the builder, optionally starting on a column.
- `hideInChooser: true` on a column keeps it out of the column chooser, the Field chooser entry of the header menu and the side bar's Columns panel. Search, filters and export still read it.
- `data-kanuni-portal` says what a popup layer is: `popup` for a menu or list the user works in, `tooltip` for a hint, `host` for the wrapper that `portalContainer` adds.

### Fixed

- On a text column the `in` and `=` conditions turned both sides into numbers: an empty string, a space and `0` matched each other and `012` matched `12`. They compare as text now, in the main thread and in the worker.
- In the header filter of a column holding both null and empty strings, ticking Blanks selected whichever of the two was seen first. Each is its own row and its own condition now.
- On a large client-side column, whose list is prepared in background slices, the header filter listed the stored value of a cell that a `valueFormatter` or a lookup draws differently; it lists what the cell shows, as the smaller lists did.
- Tab left the header filter popup and went on to the page behind it; it now goes round inside the popup. A click on an empty part of the popup no longer drops the focus, and typing there goes to the search box.
- After Escape closed a header filter that the mouse had opened, the focus went back to the filter button even when it was hidden, and was lost; it goes to the header cell then.

## DataGrid v1.10.3 — October 23, 2026

`@kanunilabs/datagrid-core@1.10.3` · `@kanunilabs/datagrid@1.9.3` · `@kanunilabs/datagrid-react@1.8.3` · `@kanunilabs/datagrid-enterprise@1.10.3` · `@kanunilabs/datagrid-react-enterprise@1.9.3` · `@kanunilabs/datagrid-server@1.1.1`

_The header filter lists every value of a server-side column, the toolbar's parts are yours to pick, and popups work inside a modal._

### Before you upgrade

- The toolbar search runs at once when the box loses focus or on Enter, instead of only after the debounce. Pass `toolbar={{ search: { applyOnBlur: false } }}` for the old timing.
- Number editors inside cells are right-aligned and have no spin buttons; editors keep the cell's padding, so the value stays where the cell drew it.
- In the header filter of a client-side list, Select all and Clear act on the values the search box shows when it has text (as in Excel); ticks the search hides are left as they were. With the box empty they act on the whole list, as before. Before, Select all ticked every value whatever the search said.
- `FilterOperator` gains `notin`. A TypeScript `switch` over the operators that has to be exhaustive needs a case for it; a `notin` condition in a filter you send to your own server is the complement of `in`, blanks included.
- A server-side source keeps working unchanged: a `loadDistinctValues` that answers an array still gets the single list of at most `limit` values. Answering a page (`{ values, totalCount, hasBlanks }` to a request with `skip` and `take`) is what lists every value.
- The Enterprise status bar is no longer `role="status"` itself: its counts sit in an inner `.kanuni-datagrid-status-live` element that carries the role, and your `itemsAfter` moved into its own `.kanuni-datagrid-status-items--end` group outside it. Tests or CSS that looked for `[data-testid="dg-status-bar"][role="status"]` should look for the inner element.

### Added

- `toolbar` takes an object as well as `true` / `false`: `search`, `columns` and `export` pick its parts, each on by default, so `toolbar={{ export: false, columns: false }}` is the search box alone.
- Search box settings under `toolbar.search`: `placeholder`, `maxLength`, `debounce`, `applyOnBlur` and `validate`, whose message refuses a text: it is shown beside the box and never searched, on a server-side grid never sent.
- `GridToolbarButton` (React) and `toolbarButton()` (plain JavaScript): an app's own icon button with a name, a shortcut hint and, while it cannot be used, the reason, for the toolbar or the status bar.
- `portalContainer` picks where the grid draws its popups (header filter, menus, date and lookup lists, dialogs): an element, or a function returning one. Put them inside a modal such as a Radix Dialog, whose `pointer-events: none` on the rest of the page made a popup on `<body>` unclickable. Popups stay under their control even when the modal is centred with a CSS `transform`, and every popup carries `data-kanuni-portal` so your own click handling can recognise it.
- `statusBar.itemsBefore` puts your items at the start of the status bar; `itemsAfter` is its end.
- The header filter on a server-side grid can list every value of a column. A source that answers a paged `loadDistinctValues` (`skip`, `take`, `search` in, `{ values, totalCount, hasBlanks }` out) gets a list as long as the server's, loaded as it is scrolled (a jump to the end asks for the last piece only), searched on the server, with the blanks as a row of their own. Select all, Clear and the ticks work on the whole list, with a search active and with pieces not loaded: the selection is kept as everything except these or only these, and becomes one `notin` or `in` filter, however long the list.
- `remote.distinctFollowsSearch` makes that list follow the grid: the request carries the other columns' filters and the quick search, and is asked again when they change. Off by default; the list is the whole column.
- The `notin` filter operator, the exact complement of `in` (blank cells pass), in the client engine, the worker and the SQL builder.
- `@kanunilabs/datagrid-server` answers the paged request (a page, a count and a blank probe over one set of conditions; `?distinct=` takes `skip`, `take`, `search`, `filter`, `searchText`), `createRestDataSource` sends it, and `notin` is a SQL operator there.

### Fixed

- In the form and popup editors a field the user cannot change looked like one they could; it is now drawn recessed, with muted text and a not-allowed cursor. The date field there takes the form's box instead of the in-cell underline.
- In row editing the focused cell kept its focus ring around the editor, and number values sat to the left of their column.
- A screen reader read the status bar's buttons again every time the row count changed: the whole bar was one live region. Only the counts are now.
- The header filter popup was torn out of the page and rebuilt about 20 ms after opening, when its values arrived, and the keys typed in between were lost. It is one mount from opening on now, and takes the values and ticks when they land.

## DataGrid v1.10.2 — October 20, 2026

`@kanunilabs/datagrid-core@1.10.2` · `@kanunilabs/datagrid@1.9.2` · `@kanunilabs/datagrid-react@1.8.2` · `@kanunilabs/datagrid-enterprise@1.10.2` · `@kanunilabs/datagrid-react-enterprise@1.9.2`

_Each row shows whether its save is pending, running, done or refused, in the free grid._

### Before you upgrade

- With `rowStatus.messages` on, the rows the grid draws include a row of kind `message` under each refused save. Code that walks the displayed rows and switches on `kind` should skip it; with the option off nothing changes.
- Every palette and colour scheme defines three new tokens, `--dg-success`, `--dg-success-text` and `--dg-on-success`. A theme of your own that sets every `--dg-*` token keeps working and can add them.

### Added

- `rowStatus` marks each row's save status (changed, saving, saved, refused) in React and plain JavaScript, free: a marker on the row's leading edge, an optional tint, and the status read out by screen readers. It follows the grid's own saves in every editing mode; `get` lets the app report rows it saves itself.
- `rowStatus.icons` adds a narrow column at the start of each row with a glyph per status (dot, spinning ring, tick, triangle), so the status does not rest on colour alone. The ring stands still under reduced motion.
- `rowStatus.messages` puts a refused save's reason under its row, with Retry and Delete row. `errors` supplies reasons from the app (by row key) and replaces the grid's own text; `onRetry` retries the app's own saves; Delete row shows only when `onDelete` is given, and the app does the deleting.
- `--dg-font-family` sets the grid's typeface, menus opened on the page included when it is set on `:root`; unset, the grid inherits the page font as before. `--dg-success`, `--dg-success-text` and `--dg-on-success` colour success states.

## PivotGrid v1.11.1 — October 20, 2026

`@kanunilabs/pivotgrid-core@1.11.1` · `@kanunilabs/pivotgrid@1.6.1` · `@kanunilabs/pivotgrid-react@1.8.1` · `@kanunilabs/pivotgrid-enterprise@1.7.1` · `@kanunilabs/pivotgrid-react-enterprise@1.8.1`

_Palette, dark mode and typeface options, so a PivotGrid matches the DataGrid beside it._

### Before you upgrade

- Numbers in the data cells use tabular figures (every digit the same width), so columns of amounts line up. Text in a fixed-width font looks the same; in a proportional font the numbers can take slightly more room.

### Added

- `palette` and `dark` on both renderers pick a colour scheme and its dark variant, the same 36 schemes the DataGrid offers; `dark` alone gives the DataGrid's dark look.
- `--pivot-font-family` sets the typeface (unset, the page font is inherited as before), and `--pivot-success-bg` / `--pivot-success-foreground` colour success states. A theming guide lists every variable.

## DataGrid v1.10.1 — October 16, 2026

`@kanunilabs/datagrid-core@1.10.1` · `@kanunilabs/datagrid@1.9.1` · `@kanunilabs/datagrid-react@1.8.1` · `@kanunilabs/datagrid-enterprise@1.10.1` · `@kanunilabs/datagrid-react-enterprise@1.9.1`

_Batch, row, form and popup editing are now in the free DataGrid._

### Before you upgrade

- The free `DataGrid` and `createGrid` now honour `editing.mode`: `batch`, `row`, `form` and `popup` work instead of falling back to cell editing. An app that passed one of them to the free grid gets that mode now. In these modes the grid does not write into your rows: `onSave` receives the changes and the app writes them into its data.
- The styles for the Save / Cancel strip, changed and removed cells, the edit form and the popup moved from the Enterprise stylesheet to the free one. Load both stylesheets, as the setup guide says; an Enterprise app that loaded only `@kanunilabs/datagrid-react-enterprise/styles.css` (or `@kanunilabs/datagrid-enterprise/styles.css`) loses them.
- Enter on a button of the edit form (Cancel, for instance) now presses that button; it used to save the row. Grid columns such as the selection check box are no longer offered as form fields.
- In the free React grid a cell that is still being saved in cell mode carries the changed-cell marker, as the plain-JavaScript grid already did.

### Changed

- Batch, row, form and popup editing moved from Enterprise to the free edition, in React and plain JavaScript: the Save / Cancel strip, every cell of a row opened at once (double-click, or Edit row… in the row menu), the form in the row flow and the form in a dialog, with `formLabels` and `formHeight`. Validation rules, CRUD handlers, adding and deleting rows, per-row permissions, lookup and custom editors, undo and redo, range selection and the command column stay Enterprise.
- `GridEditForm` (React) and `buildEditForm` (plain JavaScript) are exported from the free packages; the Enterprise packages still export them.

### Fixed

- Enter on the edit form's Cancel button saved the row instead of cancelling it, and in the plain-JavaScript grid Tab on a form button left the form.
- In the row, form and popup modes, F2 or typing on a focused cell opens the row (in `row` mode with the typed character in that cell). It used to open a lone cell editor whose value was never saved.
- In row mode a check-box column is edited with a check box in React too (it was a text box that saved "true" as text), and the form leaves out the grid's own columns such as the group column.

## DataGrid v1.10.0 — October 15, 2026

`@kanunilabs/datagrid-core@1.10.0` · `@kanunilabs/datagrid@1.9.0` · `@kanunilabs/datagrid-react@1.8.0` · `@kanunilabs/datagrid-enterprise@1.10.0` · `@kanunilabs/datagrid-react-enterprise@1.9.0` · `@kanunilabs/datagrid-server@1.1.0`

_State restore for tabbed apps, columns that give way on a narrow screen, and a REST grid that no longer leaves rows blank._

### Before you upgrade

- `createRestDataSource` cancels a running request only when the query changes (sort, filter, search, grouping). Blocks of one query now load side by side, so a server sees a few requests at once while the reader scrolls; before, each new block cancelled the one before it and rows could stay blank.
- The column chooser lists every column with a check box, under a "Columns" title, with a search box above eight columns; before, it listed only the hidden ones and opened empty when none was hidden. The last visible column cannot be hidden.
- "Chart by this column" on a number column charts it as a series (its values summed per category) instead of using every amount as a category. "Show group footers" appears in the header menu only while the grid is grouped.
- A cell with a search highlight holds its text and marks in one `span.kanuni-datagrid-cell-text`. CSS that styled the cell's direct children should target that span.
- Toolbar and chart-panel commands carry an icon beside their label (`svg.kanuni-datagrid-icon` + a `span`), and the chart panel's commands sit in `.kanuni-dg-chart-actions`. Code that read a button's `textContent` still gets the label.
- In batch editing the Save / Cancel strip is not mounted while nothing is pending; before, `.kanuni-datagrid-filter-bar` was an empty band. Tests that looked for it right after mount should wait for a change.
- A detail panel is as wide as the viewport, not as wide as all the columns, and pressing its arrow no longer selects the row.
- In development the console warns once when a grid has no height of its own and draws every row of a large result.
- `@kanunilabs/datagrid-server`: an unknown sort direction is now a 400 (`SqlRequestError`) instead of being read as ascending, and on PostgreSQL the nils-last sort is written `col ASC NULLS LAST`. An expression index you built on the old `CASE WHEN col IS NULL …` term no longer matches there; `indexStatements` writes the ones that do.

### Added

- `hideBelow` on a column hides it while the grid is narrower than that many pixels, as a layer of its own: the saved view keeps what the user chose, the column chooser shows it ticked, and it comes back by itself on a wide screen. Cards leave it out too.
- Restoring into a rebuilt grid: `getViewState({ include: ['details'] })` keeps the open detail panels, and a restored focus on a server's row waits for its block instead of being dropped.
- `@kanunilabs/datagrid-server`: `indexStatements(config, { columns })` writes the indexes the generated SQL can use (PostgreSQL, MySQL 8, SQLite), and `nullable: false` on a column drops the NULL handling from its sort and its `=` / `<>`.
- A read-only grid that still selects cells and ranges and copies them: `editing: { enabled: false, range: true }` keeps the range and Ctrl+C (the text the cells show, as TSV) while nothing can be edited, pasted, cut or filled (Enterprise).
- Icons on the toolbar's Columns and Chart buttons and on the chart panel's commands; the chart types read as one control. `ToolbarIcon` is exported from the React package.

### Fixed

- A REST grid scrolled faster than its server answered left rows as skeletons for good once the server took longer than about 130 ms per block.
- A search highlight dropped the spaces next to the match ("HEARTMEASURING" for "heart measuring").
- Under React's StrictMode a custom `stateStoring` store was read three times for one grid; it is read once.
- The field chooser opened nothing when no column was hidden.

## PivotGrid v1.11.0 — October 15, 2026

`@kanunilabs/pivotgrid-core@1.11.0` · `@kanunilabs/pivotgrid@1.6.0` · `@kanunilabs/pivotgrid-react@1.8.0` · `@kanunilabs/pivotgrid-enterprise@1.7.0` · `@kanunilabs/pivotgrid-react-enterprise@1.8.0`

_The whole state in one piece for tabbed apps, Count Distinct that counts text, and charts that show what the grid shows._

### Before you upgrade

- `exportData(format)` without `applyFilters` now writes only the rows the filters keep, as the footer's export buttons always did. Pass `applyFilters: false` for every source row.
- Restoring a saved layout drops the fields your app no longer defines (the calculated fields a reader made stay), and fields added since then join as you define them. Before, a removed field came back without its data field.
- The saved state carries `version: 1`; a state with another version is ignored and the grid opens with its layout. States saved before this version have none and still load.
- A field counted distinct leaves blank cells out of the count, the way the main-thread engine and the server pivot always did; before, a blank counted as one more value on a number field.
- Under a display mode (% of total, running total…) the chart draws the values the grid shows instead of the sums under them, and a percent measure prints as a percent in tooltips and on the axis.
- The dictionary gained `distinct` ("Count Distinct"). A TypeScript app that types a complete dictionary of its own adds the key. The Enterprise prefilter builder's eight captions (`builderCombine`, `builderAddCondition`, `builderAddGroup`, `builderRemoveGroup`, `builderRemoveCondition`, `builderEmpty`, `opIsBlank`, `opIsNotBlank`) left the shared dictionary type: overrides of them still apply at runtime, but TypeScript now reports them as unknown keys in a dictionary typed as `Partial<PivotDictionary>`; keep them in an object of their own type.

### Added

- `getPersistedState()` and `applyPersistedState(state)` on the React ref and the controller: everything the reader arranged as versioned plain JSON, for a tab you unmount and rebuild or a view kept on your server.
- `stateStoring.onLoad` may return a Promise (IndexedDB, a request); nothing is saved until it lands, so the default layout never overwrites the state being loaded.
- Parts registered with `registerReportStatePart` (the Enterprise chart panel's chart, or your own) are saved with the state; `notifyStatePartChanged()` saves a change. `include: ['scroll']` keeps the body's scroll, put back once the pivot is drawn.
- Count Distinct in the Field Settings dialog: text and date fields offer Count and Count Distinct, number fields list it after the others, in twelve languages.

### Fixed

- A distinct count of a text field (customers, order numbers) read 1 in every cell.
- CSV and Excel list export stopped with a DataCloneError when a field's `format` was a function.
- A field the app added after a layout was saved was missing for every returning user.
- The React chart cut long category labels at a fixed 60 px strip and wrote large axis values as "14000k".

## DataGrid v1.9.0 — October 14, 2026

`@kanunilabs/datagrid-core@1.9.0` · `@kanunilabs/datagrid@1.8.0` · `@kanunilabs/datagrid-react@1.7.0` · `@kanunilabs/datagrid-enterprise@1.9.0` · `@kanunilabs/datagrid-react-enterprise@1.8.0` · `@kanunilabs/datagrid-server@1.0.0`

_Free cell editing, cards on narrow screens, your own filter operators, and a ready-made SQL server package._

### Before you upgrade

- A footer or group total on a number column with no formatter now groups its digits in the grid's locale and keeps at most two decimals (`260,064,641.94` instead of `260064641.94`). A column or summary formatter still decides when you set one.
- A filter condition whose operator the column does not know now matches no rows; before, it let every row through. A saved view that names an operator the column no longer registers is the likely source, and `debugMode` names it on the console.
- `createRestDataSource` now sends `summaries` and `parentKey` in the query string, JSON-encoded like `sort` and `filter`, when the grid asks for footer totals or a tree level. A server that refuses unknown parameters has to accept them.
- With grouping on the server, `summaries` now comes with the load request of every grouping level, not only the top one, so sub-group totals can be filled (they stayed empty before). The footer is still read from the answer for the top level only (Enterprise).
- An edit on a column whose `id` differs from its `field` now writes the row's `field`; before, it wrote a property named after the `id`.
- In the Enterprise grid a refused `onSave` now shows its error dialog; before, the dialog was cleared before it could appear.
- The plain-JavaScript grid lays out right to left for Arabic, as the React grid already did.

### Added

- Cell editing is free: `editing` on the Community React grid and `createGrid({ editing: {} })` in plain JavaScript open an in-cell editor on double-click, F2 or typing. Enter commits and moves down, Tab moves on, Esc cancels, and `onSave` gets each committed cell. Edit modes, validation rules, adding and deleting rows, undo, ranges, fill and paste stay Enterprise.
- Cards on narrow screens: `cardView` draws each row as a card (a title, then labelled fields) when the grid is narrower than a breakpoint (600px by default), with a Sort by control in place of the headers. Sorting, filtering, selection, grouping, server data and virtual scrolling keep working; editing stands down while cards are shown, and `onCardViewChange` tells you when the grid switches.
- Your own filter operators: a column's `filterOperators` (`{ id, label, test, operandType }`) appear in the operator menu and in the Enterprise filter builders, and `renderFilter` puts your own control in the filter cell. The condition stays plain data, so saved views and a server carry it unchanged.
- `valueParser` turns typed and pasted text into the stored value before validation runs, and `valueSetter` writes it for nested or immutable data.
- `wrapHeaderText` on a column breaks a long caption onto more lines, and `autoHeaderHeight` lets the header grow to the tallest caption.
- `@kanunilabs/datagrid-server`, a new free package: it turns the grid's load request (sort, filter, quick search, paging, footer totals, server grouping and tree levels) into parameterized SQL for PostgreSQL, MySQL, SQLite and SQL Server, with the grid's own filter meaning. It has no dependencies; you pass the function that runs a query.
- `createRestDataSource({ distinctValues: true })` offers the header filter checklist from the same endpoint.

### Changed

- A number total with no formatter groups its digits in the grid's locale, like the row count beside it.
- An operator id the column does not register matches no rows instead of every row.

### Fixed

- With fewer than 20,000 rows, the React grid could stay on "No data" after it mounted, in every version since 1.0.0: the first view was computed before the component had subscribed to it, and the update was lost. The rows now appear on first paint. Larger datasets, which go through the Web Worker, the plain-JavaScript grid and PivotGrid were not affected.
- A REST grid's footer stayed empty and a server tree could not tell which level was asked for: `summaries` and `parentKey` were never sent, and returned `summaries` were dropped.
- Accessibility: the filter row's cells are grid cells; the confirm and error dialog keeps Tab inside it, as `aria-modal` promises, with ids unique per dialog; the focus ring, selection and editor stay visible in Windows high-contrast mode.
- The Enterprise grid never showed the dialog for a refused save.

## PivotGrid v1.10.0 — October 14, 2026

`@kanunilabs/pivotgrid-core@1.10.0` · `@kanunilabs/pivotgrid@1.5.0` · `@kanunilabs/pivotgrid-react@1.7.0` · `@kanunilabs/pivotgrid-enterprise@1.6.0` · `@kanunilabs/pivotgrid-react-enterprise@1.7.0`

_Quick search over the rows, a cap on runaway columns, and accessibility fixes._

### Before you upgrade

- A pivot with more than 1,000 column members now draws the first 1,000 and says so above the table; the totals still count all the data. Set `maxColumns` to raise the cap, or `0` to turn it off.
- The filter and remove icons on a field chip are no longer buttons, because a control inside the chip's own button cannot be reached as a second control by a screen reader. Mouse clicks work as before; from the keyboard, Alt+ArrowDown on the chip opens its filter and Delete removes it. A test or script that selected `button.kanuni-pivot-chip-btn` should select `.kanuni-pivot-chip-btn`.
- The summary badge on a value field (SUM, AVG…) is drawn in the text colour on an accent tint with an accent border, instead of accent-coloured text.

### Added

- Quick search: `search` puts a box in the footer that shows only the row members whose caption contains the text, with the members above them, on top of the reader's own filters; `setSearchText(text)` does the same from code. Nothing is saved with the layout, so clearing it gives the view back.
- `maxColumns` (1,000 by default): a field with a different value on almost every row no longer freezes the page when it is dropped on the columns. `getEngineResult().columnsTruncated` reports `{ shown, total }`. Exports are not cut: they keep every column.
- Alt+ArrowDown on a field chip opens its member filter, announced in the chip's key description in twelve languages.

### Fixed

- The pivot grid has an accessible name, and keeps its focus ring and selection visible in Windows high-contrast mode.
- The summary badge on a value field measured 2.77:1 contrast, below the 4.5:1 that small text needs.

## DataGrid v1.8.0 — October 12, 2026

`@kanunilabs/datagrid-core@1.8.0` · `@kanunilabs/datagrid@1.7.0` · `@kanunilabs/datagrid-react@1.6.0` · `@kanunilabs/datagrid-enterprise@1.8.0` · `@kanunilabs/datagrid-react-enterprise@1.7.0`

_36 colour schemes and a Theme Builder, flex columns, row animation, rich tooltips, and a viewport row model._

### Before you upgrade

- Four structure themes are renamed: `quartz` is now `minimal`, `balham` is `ledger`, `material` is `elevated` and `alpine` is `framed`, and their `kanuni-datagrid-theme-*` classes with them. An old name is a type error in TypeScript; in plain JavaScript it is treated as a theme of your own and falls back to the default look.

### Added

- 36 colour schemes, light and dark, on the `palette` prop, each measured against WCAG AA contrast and shipped as its own stylesheet (`themes/<id>.css`, `themes/all.css`). The free Theme Builder on kanunilabs.com/theme-builder designs your own and exports the CSS.
- Flex columns that share the grid's free width, exact filtering and sorting of integers beyond 2^53 with a `bigint` column type, pinning a row from its menu, column spans (`colSpan`) and row animation (`animateRows`).
- Tooltips that open on keyboard focus and on touch, can be interactive and can hold your own content.
- Find without filtering with Ctrl+F, selecting a whole column from its header, renaming a column inside the grid, clipboard callbacks, your own side panels, and editing a group total that is spread over its rows (Enterprise).
- A viewport row model: the server pushes only the rows on screen, and their changes (Enterprise).
- Charts: named series palettes in the Format dialog — Classic, Colour-blind safe, Vivid, One colour — kept at 3:1 contrast with the chart background, and `colors` for your own list (Enterprise).

### Changed

- Four structure themes are renamed: `minimal`, `ledger`, `elevated`, `framed`.
- A range chart with one value column says it needs two instead of drawing empty axes (Enterprise).

### Fixed

- A group's subtotal stayed stale after a cell under it was edited.
- In React, the range anchor cell lost its custom renderer.

## PivotGrid v1.9.0 — October 12, 2026

`@kanunilabs/pivotgrid-core@1.9.0` · `@kanunilabs/pivotgrid@1.4.0` · `@kanunilabs/pivotgrid-react@1.6.0` · `@kanunilabs/pivotgrid-enterprise@1.5.0` · `@kanunilabs/pivotgrid-react-enterprise@1.6.0`

_36 colour schemes, row density, chart palettes, and the grand-total flags read the way they are documented._

### Before you upgrade

- `showRowGrandTotals` now controls the grand total row and `showColumnGrandTotals` the grand total column, as documented; before, each switched the other — in both renderers, the exports and the clipboard. An app that turned off only one of them sees the other one hidden now; if you had swapped the two to get the right result, swap them back.
- In the header filter list a tick always means "shown", in include and exclude mode alike. A saved filter is read the same; only what the list displays changed.
- The footer toolbar buttons get the padding they were meant to have (a 27px target at normal density), so the toolbar is a little taller than in 1.8.0.

### Added

- 36 colour schemes, light and dark, as stylesheets next to the grid's own (`themes/<id>.css`), named by `theme={{ id }}`; they reach every popup and dialog. The theme object gains `headerBackground` and `headerText` for a header band apart from the panels.
- Row density: `density` `'compact' | 'normal' | 'comfortable'` (24, 28 or 36px rows), and `setDensity()` in plain JavaScript.
- Charts: named series palettes — Classic, Colour-blind safe, Vivid, One colour — kept at 3:1 contrast with the chart background, and `colors` for your own list; a Colours picker in the JavaScript chart's Format dialog, and the names on `ChartFactory`'s `palette` (Enterprise).

### Changed

- The rows behind a clicked cell are found when `records` is first read, matching the cell's path first: 31–62 ms per click on a million rows, down from 300–490 ms.
- A range chart with one measure says it needs two instead of drawing empty axes (Enterprise).

### Fixed

- The grand-total flags were swapped everywhere: the row flag drew the column and the other way round.
- Top N "Others": its outer row was empty and its summary added up instead of applying the summary formula.
- The header filter list showed ticks the other way round in exclude mode, and the field-settings and save-view windows were laid out wrongly in React.
- The footer toolbar buttons painted as bare 15px icons with no hover border.
- The React `locale` prop was documented as defaulting to Turkish; it defaults to English.

## DataGrid v1.7.0 — October 9, 2026

`@kanunilabs/datagrid-core@1.7.0` · `@kanunilabs/datagrid@1.6.0` · `@kanunilabs/datagrid-react@1.5.0` · `@kanunilabs/datagrid-enterprise@1.7.0` · `@kanunilabs/datagrid-react-enterprise@1.6.0`

_Formulas, server-side data in the free edition, a separate group column, and much more for charts._

### Before you upgrade

- A plain click on a chart category keeps every category on the chart, the others dimmed; before, it narrowed the chart to the clicked one. Ctrl/Cmd-click works as before.
- A remove and an add of the same row id in one `applyTransactionAsync` now replace the row; before, they cancelled each other out.
- Loading rows from a server no longer needs a licence. On a Community grid fed by a server, the grouping, server tree and whole-result export entries are not offered; those stay Enterprise.

### Added

- Formulas. A column can be a spreadsheet formula over its row (`formula: '[Price] * [Qty]'`, Community); with `calculatedColumns` your users add, edit and remove their own from the column menu, with a live preview (Enterprise); and with `allowFormulas` a cell can hold `=A1*B2`, kept on the same cells through sorting and filtering (Enterprise). No `eval`, so it runs under a strict Content-Security-Policy.
- Excel exports write calculated columns and cell formulas as live Excel formulas, and a column's `exportLink` makes its cells hyperlinks in Excel and PDF (Enterprise).
- Master-detail details in Excel and PDF exports: each master row's detail table under it, from `masterDetail.export` (Enterprise).
- A separate group column: `grouping.display: 'column'` puts the group names in their own column instead of a banner.
- Charts: floating windows and several charts per grid, a chart in your own container (`charting.container`), detach and reattach, a chart from a selected cell range painted in the grid, text, label angle, grid and series line options, an axis label formatter, a calendar date format, `first` / `last` aggregations, a column's chart role, cross-filtering on several categories, and category selection from the keyboard (Enterprise).
- Sparklines: horizontal bars, `[x, y]` and object data on a number or time axis, line and area styles, markers, value labels, your own colours and a multi-line tooltip.
- Sorting: by magnitude (`absoluteSort`), the click cycle per column, and a hook after every sort.
- Rows that cannot be selected (`selection.isRowSelectable`), numbered page buttons (`pagination.pageButtons`), and several cell ranges with Ctrl/Cmd (Enterprise).
- Server-side data: rows that fail to load say so with a Retry button, `invalidate({ background: true })` refreshes without the loading overlay, and Expand all opens server groups as they arrive.
- Saved views also keep footer aggregations, group footers, folded column bands and open server groups; selection, focus, scroll and cell ranges on request (`include`).
- Accessibility: the grid name (`ariaLabel`), named tree and band toggles, and in the JavaScript grid keyboard access to headers, filters and menus and announced size and selection.

### Changed

- Loading rows from a server is now in the Community edition. Grouping on the server, a server tree and an export of the whole result stay Enterprise, and are not offered on a Community grid fed by a server.
- A plain click on a chart category keeps the others visible, dimmed.
- The loading overlay also shows while a slow recompute the grid itself started is running.

### Fixed

- A refresh of server data that failed emptied the grid: a flat grid turned every row into a Retry row and a server-grouped one collapsed to a single row. The rows now keep their last data.
- `applyTransactionAsync` cancelled a remove and an add of the same row in one transaction instead of replacing the row.
- Saved views were not written after collapsing a group or changing the page or page size, and a view restored onto a server-grouped grid while its first load ran could stall on one loading row.
- The group panel's Expand all / Collapse all were English in every language.
- The chart ignored an external filter on the Web Worker path.
- In the JavaScript grid, Enter and Space did not press the numbered page buttons.

## PivotGrid v1.8.0 — October 9, 2026

`@kanunilabs/pivotgrid-core@1.8.0` · `@kanunilabs/pivotgrid@1.3.0` · `@kanunilabs/pivotgrid-react@1.5.0` · `@kanunilabs/pivotgrid-enterprise@1.4.0` · `@kanunilabs/pivotgrid-react-enterprise@1.5.0`

_Value and label filters, live updates in batches, a histogram, and fewer pauses in React._

### Added

- Value and label filters: keep members by their totals or their names, from code or the header menu (Community); the AI state schema carries them (Enterprise).
- `applyTransactionAsync`: a fast stream of updates is applied in batches instead of one recompute each.
- Date grouping by ISO week, and percent of the parent row or column total.
- A histogram chart, floating chart windows, several charts per grid, detach and reattach, and category selection from the keyboard (Enterprise).
- Accessibility: the tree layout is a tree grid with row levels, sort and filter changes are announced, the focused element is the one a screen reader follows, and the React drill-down window is a real dialog in the grid's language.

### Changed

- React: a wide pivot draws the visible cells once per change instead of three or four times — far fewer pauses while scrolling and filtering.
- The free JavaScript package is about 21 KB smaller.

### Fixed

- On a page with several pivot grids, one grid could drive another's engine.
- A bare ISO date was read a day early west of UTC.
- The user's display mode and date range were lost on a refresh, and a pending saved view was not written when the grid was destroyed.
- Changing only Top N or "Others" did not recompute.
- Percent displays on an unformatted field showed plain numbers.
- Ctrl+C threw on a page without clipboard access.
- The React "expand all" icon was drawn closed.

## PivotGrid v1.7.2 — October 8, 2026

`@kanunilabs/pivotgrid-core@1.7.2` · `@kanunilabs/pivotgrid@1.2.2` · `@kanunilabs/pivotgrid-react@1.4.2` · `@kanunilabs/pivotgrid-enterprise@1.3.2` · `@kanunilabs/pivotgrid-react-enterprise@1.4.2`

_Wide pivots stop freezing on load; a partial layout no longer empties the grid._

### Before you upgrade

- A partial `layout`, such as `{ rowHeaderLayout: 'tree' }`, keeps the defaults for the options it leaves out; before, the grid drew no values. A full set of defaults passed only to work around that can go.

### Fixed

- Sorting the row and column headers built a new locale comparer for every comparison, so a pivot with thousands of members froze for about half a second on load and on every re-sort. One comparer per locale is now reused: 5,000 members sort in 12–25 ms instead of 333–386 ms. The order is unchanged. The member filter list sorts the same way.
- A `layout` that named only some options, such as `{ rowHeaderLayout: 'tree' }` or even `{}`, replaced the defaults instead of landing on them, and the grid drew no values. The options you leave out now come from the defaults. In the JavaScript and the React grid.

## DataGrid v1.6.0 — October 6, 2026

`@kanunilabs/datagrid-core@1.6.0` · `@kanunilabs/datagrid@1.5.0` · `@kanunilabs/datagrid-react@1.4.0` · `@kanunilabs/datagrid-enterprise@1.6.0` · `@kanunilabs/datagrid-react-enterprise@1.5.0`

_Excel exports that Excel opens past a million rows, with group subtotals and real dates; sticky group headers; fill from the keyboard._

### Before you upgrade

- Exports format the grand-total text the way the grid shows it: a PDF totals row reads "$50.00" where it read "50".
- Excel exports write ISO date strings and epoch numbers in a date column as real dates instead of text.
- A date column with a `dateFormat` exports and copies the formatted date (CSV, PDF, clipboard, filter list) instead of the stored value.

### Fixed

- Excel export: a sheet holds 1,048,576 rows, header included, and nothing counted them. One row more and Excel refused the file, or repaired it by deleting every row past the limit, with only a line in a dialog to say so. Rows past the limit now continue on "Data (2)", "Data (3)"…, each with the header, a filter and a frozen top row, so every row arrives in a file Excel opens. Measured in Excel: 1,067,371 rows open in about 6 s on two sheets.
- Exports carry group subtotals. With `groupSummaries`, a group row exported only its label and count; each subtotal now sits under its own column, formatted as the grid shows it and written to Excel as a number. In CSV, Excel and PDF.
- Excel export writes a date column as dates. An ISO string such as "2011-11-01", which is how a JSON API sends a date, was written as text that Excel's date filter and sort ignore; it is now a real date, as is an epoch number. A local-midnight `Date` no longer lands on the previous day in time zones east of UTC: Excel shows the calendar day and clock time the grid shows.
- A date column with a `dateFormat` reads the same in CSV, PDF, copied cells and the filter checklist as it does in its cells, instead of the stored ISO string or a Date's long form.

### Added

- `grouping.stickyHeaders`: the headers of the groups you are scrolling through stay under the column header, outermost first, and the next group pushes them out. Off by default.
- `maxRowsPerSheet` on Excel export, to start a new sheet sooner than Excel's limit.
- Enterprise: Ctrl/Cmd+D fills down and Ctrl/Cmd+R fills right, copying the first row or column of the selected range, or the cell above or to the left of a single cell. They follow the same permissions as the fill handle and undo as one step.

### Changed

- The text of the grand total in an export now uses the grid's formatting, the same as the footer and the new group subtotals. A PDF totals row reads "$50.00" where it read "50".

## PivotGrid v1.7.1 — October 6, 2026

`@kanunilabs/pivotgrid-core@1.7.1` · `@kanunilabs/pivotgrid@1.2.1` · `@kanunilabs/pivotgrid-react@1.4.1` · `@kanunilabs/pivotgrid-enterprise@1.3.1` · `@kanunilabs/pivotgrid-react-enterprise@1.4.1`

_The packages load under plain Node ESM._

### Fixed

- Importing a PivotGrid package as ESM in plain Node (server rendering, scripts, a test runner in a Node environment) failed with "Named export 'saveAs' not found", because of how the packages imported file-saver. Bundlers were not affected. Every package build now checks that its output loads this way.

## DataGrid v1.5.0 — October 5, 2026

`@kanunilabs/datagrid-core@1.5.0` · `@kanunilabs/datagrid@1.4.0` · `@kanunilabs/datagrid-react@1.3.0` · `@kanunilabs/datagrid-enterprise@1.5.0` · `@kanunilabs/datagrid-react-enterprise@1.4.0`

_Tree data from paths, nested arrays or a server; calendar grouping and group selection; relative date filters; many more chart types; and a deferred select-all that reports what it selected._

### Before you upgrade

- With deferred select-all, `selectionChanged` reports the selected rows and their count; before, it reported no rows and a count of 0.

### Added

- Tree data in two more shapes. `tree.getDataPath` takes each row's place as a list of names; a level with no row of its own becomes a folder row that opens, closes and sorts like any node but is not a record (it cannot be selected or edited). `tree.childrenField` reads nested `children` arrays; edits still write to your objects.
- Server-side tree: with a remote `DataSource`, `tree.hasChildren` loads the tree level by level through `load({ parentKey })`. Opening everything loads only the levels that come into view.
- `groupInterval: 'day' | 'week' | 'month' | 'quarter' | 'year'` on a date column groups by calendar unit, with headers in the grid's language.
- `grouping.display: 'aligned'` lays a group row out as cells: the group's name in the first column and each group summary under its own column.
- A checkbox on each group header selects every row under the group, open or closed (`toggleGroupSelection`, `setGroupSelected`, `getGroupSelectionState`, `getGroupRowKeys`).
- Relative date filters: a `period` condition such as `{ period: 'thisMonth' }` or `{ period: 'lastDays', days: 7 }` is worked out each time the filter runs, so a view saved as "this month" still means this month next week. The filter row accepts period names in the grid's language or in English.
- `expandAllDetails()`, the `treeRowToggled` and `detailToggled` events, and the tree's open rows in the saved view state.
- Keyboard: Left and Right open and close tree nodes and groups. Enterprise: Shift+arrow and Ctrl/Cmd+Shift+arrow extend a cell range, and the statistics bar announces it to screen readers.
- Enterprise charts: stacked and 100% stacked; a Format editor (title, axis titles, legend position, data labels, a logarithmic value axis); X-Y scatter and bubble charts with one point per row; waterfall, funnel, treemap, heatmap, box plot, radar and range bar.
- Enterprise charts: the chart is now state you can read and restore (`GridChartState`, the `chartOpened`, `chartChanged` and `chartClosed` events, saved with the view), with an API in React too (`charting.apiRef`). Images export as PNG or JPEG at a chosen size, or as a data URL.
- Enterprise sparklines: `highlight` (first, lowest, highest point), `color`, `negativeColor` and `tooltip`. A gap in the series now stays a gap instead of joining its neighbours.
- Enterprise: `getAiSchema`, `getAiState` and `applyAiState` describe the grid's state as a JSON Schema for a language model and apply its answer, keeping the valid part and listing the rest. No model or network call is included.

### Fixed

- Deferred select-all: `selectionChanged` reported no rows and a count of 0, and after a row was excluded it reported that row as the selection. It now reports the selected rows and their count. The count also no longer subtracts excluded rows that the current filter hides, and "export selected rows only" no longer writes the excluded rows.
- Server-side grouping showed one group and loading rows on the first screen until the user scrolled. The rest of the first screen now loads straight away.
- JavaScript Enterprise with a remote data source of more than 100 rows threw on its first paint and drew no rows. Rows not yet loaded now show as loading.
- Enterprise: Shift+click after a drag did not extend the cell range.
- React Enterprise: with `editing={{ enabled: false }}`, sparkline columns showed the raw array instead of the chart.
- A remote `DataSource` with `tree` but no `hasChildren` showed a flat list without saying so. It now warns once when the grid is created.

## PivotGrid v1.7.0 — October 5, 2026

`@kanunilabs/pivotgrid-core@1.7.0` · `@kanunilabs/pivotgrid@1.2.0` · `@kanunilabs/pivotgrid-react@1.4.0` · `@kanunilabs/pivotgrid-enterprise@1.3.0` · `@kanunilabs/pivotgrid-react-enterprise@1.4.0`

_Live updates without a rebuild, a pivot your server aggregates, a trend line in each row, many more chart types, and custom summaries that really run._

### Before you upgrade

- `summaryType: 'custom'` runs your `calculateCustomSummary`, or the aggregator registered under that name. Before, every custom measure showed a sum, so the numbers of such measures change. An unknown aggregator name is reported as an engine error.
- `controller.on('fieldMove', …)` fires now. A handler registered for it starts running on every field drag; `newIndex` is where the field landed.
- `groupInterval: 'numeric'` and `'custom'` never grouped; they are now reported through `onError`.
- Fourteen chart panel captions left the `PivotDictionary` type. A dictionary declared with that type that sets them no longer type-checks; your own messages still rename them.

### Added

- Live updates: `applyTransaction({ add, update, remove })` with a `rowKey`, on the controller, `createPivotGrid` and the React `ref`. Only the changed rows are processed and only the groups they touch are summed again, with the same result as a full rebuild. Measured in Node on 1,000,000 rows: about 17 ms in the worker per update, against 230-290 ms for a rebuild.
- Enterprise: server-side pivot. `serverSource` (built with `pivotServerSource({ load, loadFieldValues })`) lets your server do the aggregating; only totals reach the browser, and a closed group is not asked for. `rollupInMemory` is the reference implementation.
- Enterprise: `rowSparkline` draws each row's values across the visible columns as a small line or bar chart in its grand total cell.
- Enterprise: `getPivotAiSchema`, `getPivotAiState` and `applyPivotAiState` describe the pivot layout as a JSON Schema for a language model and apply its answer. The React `ref` now has `getController()`.
- Enterprise charts: stacked and 100% stacked; a Format editor; X-Y and bubble charts; a logarithmic value axis; donut, waterfall, funnel, treemap, heatmap, range bar and sunburst. The chart type formerly named Scatter is now called Dot chart; saved charts still open.
- Enterprise charts: colours follow the grid's theme (`chartColors`, React `palette`); clicking a category filters the pivot (`chartCrossFilter`, React `onCategoryClick`); chart state and events (`getChartState`, `setChartState`); image export as PNG, SVG or JPEG, at a chosen size or as a data URL. In React, a second value axis, hiding a series from the legend and `maxSeries`.

### Changed

- `summaryType: 'custom'` now runs your `calculateCustomSummary`, or the aggregator you registered by name. Before, every custom measure silently showed a sum, so numbers from such a measure will change. An unknown aggregator name is now reported as an engine error.
- `controller.on('fieldMove', ...)` now fires, and its `newIndex` is where the field actually landed. Dropping a field on its own place no longer reports a move.
- Fourteen chart panel captions left the shared `PivotDictionary` type; the Enterprise chart panel carries them. Your own messages can still rename them.

### Fixed

- React: moving the last column field into the rows made the row area's field chips disappear.
- React charts: the points of a dot chart were not bound to any value, and a pie with one series was drawn in one colour.
- `groupInterval: 'numeric'` and `'custom'` did nothing without saying so. They are now reported through `onError`.

## DataGrid v1.4.2 — October 1, 2026

`@kanunilabs/datagrid-core@1.4.1` · `@kanunilabs/datagrid@1.3.2` · `@kanunilabs/datagrid-react@1.2.2` · `@kanunilabs/datagrid-enterprise@1.4.2` · `@kanunilabs/datagrid-react-enterprise@1.3.2`

_A search typed in parts prepares the data once, and a validation message is visible while you edit._

### Fixed

- On large data the first search prepares the text it searches, which takes a few seconds. Typed in parts, each new part started the same preparation again alongside the first, so "Preparing data…" stayed up longer and its percentage went backwards. The preparation now runs once, and Cancel still stops it. On our Online Retail demo (1,067,371 rows), a search typed in three parts went from 8.2 s of preparation to 2.3 s on the same machine.
- React Enterprise: a validation message raised while editing a cell was rendered but hidden under the cell and the next row, so the user saw a red underline and no reason. The message now shows.

## PivotGrid v1.6.0 — October 1, 2026

`@kanunilabs/pivotgrid-core@1.6.0` · `@kanunilabs/pivotgrid@1.1.0` · `@kanunilabs/pivotgrid-react@1.3.0` · `@kanunilabs/pivotgrid-enterprise@1.2.0` · `@kanunilabs/pivotgrid-react-enterprise@1.3.0`

_A filter on a field in the filter area is applied again, and column widths have a documented option._

### Fixed

- A field placed only in the filter area did not filter: with values selected, through `filterValues` or by the user, every total still included the excluded rows. The filter now applies, whether the values come with the initial fields, are picked after the grid is up, or survive a remount over the same data.
- React: scrolled far to the right with wide or resized data columns, some visible columns were drawn empty. The visible range is now worked out from the real column widths.
- React: a data column removed from the width map kept its old width until the grid was remounted. It now returns to the default straight away.

### Added

- `defaultColumnWidths: { rowHeader, data }` on `BasePivotGrid`, `EnterprisePivotGrid` and `createPivotGrid`: the starting width of every row-header column and every data column (defaults 150 and 95). A width the user drags still wins.
- `setColumnWidths` on the React `ref`. The keys of the width map, `row-<i>` and `leaf-<i>`, are now documented.

## DataGrid v1.4.1 — September 30, 2026

`@kanunilabs/datagrid-core@1.4.0` · `@kanunilabs/datagrid@1.3.1` · `@kanunilabs/datagrid-react@1.2.1` · `@kanunilabs/datagrid-enterprise@1.4.1` · `@kanunilabs/datagrid-react-enterprise@1.3.1`

_Saving fixes: a refused save is always shown, a new row is never lost or inserted twice, and one cancel no longer stops the grid from sorting._

### Before you upgrade

- In batch editing, a validation rule that reads another column sees the value the user just typed there, not the stored one.

### Fixed

- After a single cancel on large data, the grid stopped sorting and filtering until it was destroyed: every later request came back as cancelled. A cancel now reaches only the work that was already in flight.
- With `crud` handlers present, a change whose handler was missing was reported as saved and dropped — a row added without `crud.insert` never reached the server and vanished on reload, and a deleted row came back. Such a save is now refused before anything is sent, the changes stay pending, and the console names the missing handler.
- Two quick edits in cell mode could send the same change twice, and insert a new row twice. Saves now run one after another, each sending only what the previous one did not.
- In batch mode, a validation rule that reads a neighbouring column saw that column’s stored value instead of the one the user had just typed. Rules now see the row as edited.
- In cell mode, a save refused by a rule or by the server showed nothing, so the user believed it was saved. The grid now shows the reason, in the same dialog Save already used, and puts the user on the cell.

### Added

- `onAutoSaveFailed` on the editing engine options, for apps that drive the engine themselves and want to hear about saves the grid starts on its own.

## PivotGrid v1.5.0 — September 30, 2026

`@kanunilabs/pivotgrid-core@1.5.0` · `@kanunilabs/pivotgrid@1.0.2` · `@kanunilabs/pivotgrid-react@1.2.2` · `@kanunilabs/pivotgrid-enterprise@1.1.1` · `@kanunilabs/pivotgrid-react-enterprise@1.2.1`

_A field declared as text is never shown as a date, the React grid stays inside its container, and a JSON column named __proto__ is no longer blank._

### Fixed

- A field whose `dataType` is `string` and whose values look like “2009-12” was shown as “December 1, 2009” in the headers and “Dec 1” on the chart axis — a month read as a day. The declared type now decides: text is shown as written, `date` fields are formatted as before. Screen and export now agree.
- The React grid laid itself out against the nearest positioned ancestor on the page, so it could spread past its container and cover the component below it. It now fills exactly the element it is given; in a container with no height it takes the height of its content.
- Importing JSON with a top-level `__proto__` key produced a column that stayed blank. The key is renamed safely before the columns are discovered, as the CSV and XML imports already did.

### Added

- `formatDimensionValue` in `@kanunilabs/pivotgrid-core`: the one rule the grid, the charts and the export use to show a row or column value.

## DataGrid v1.4.0 — September 22, 2026

`@kanunilabs/datagrid-core@1.3.0` · `@kanunilabs/datagrid@1.3.0` · `@kanunilabs/datagrid-react@1.2.0` · `@kanunilabs/datagrid-enterprise@1.4.0` · `@kanunilabs/datagrid-react-enterprise@1.3.0` · `@kanunilabs/licensing@1.3.0`

_Licence keys verify on plain-HTTP intranet pages and the badge says why a key failed; Turkish-aware search; a font for PDF export; the CSV separator follows the locale._

### Before you upgrade

- The CSV separator follows the grid's `locale`: `;` for Turkish, German, French, Spanish, Italian, Portuguese, Russian and Arabic, `,` elsewhere. If a program reads your CSV files with a fixed separator, pass `delimiter` to the CSV export.
- Search, text filters and lookups treat İ, I, ı and i as one letter, and search also matches what a cell shows (`valueFormatter`, `dateFormat`). A search can find rows it did not find before.
- Types: `theme` is typed `GridThemeName`; code that copies the prop's type into a `GridTheme` should use `GridThemeName`. A dictionary typed as the full `GridDictionary` needs the eight new licence keys; a partial `messages` object needs nothing.

### Added

- Licence keys now verify on pages served over plain `http://` — an intranet host, an IP address. Browsers only offer `crypto.subtle` on HTTPS and `localhost`, so such a page marked a valid key “Invalid license”. Where WebCrypto is missing, a bundled JavaScript implementation of the same ECDSA P-256 check runs instead: same verdict, the signature is always verified, and it loads only on those pages (its own chunk in bundler builds, built into the script-tag builds).
- The licence badge says why a key failed: expired, does not cover this version, not valid for this domain, revoked, malformed, invalid signature, or “Browser cannot verify the license (HTTPS required)” — in the grid’s language, all twelve; in Arabic it sits on the left and reads right to left. The licence state carries a matching `reasonCode` for code that wants to act on it.
- `selection.rowClick` (`replace` or `toggle`): in checkbox mode a row click still toggles by default; `replace` makes it select only that row.
- PDF export takes a `font` — a TrueType file embedded in the document. The built-in PDF fonts cannot draw letters outside Latin-1 (ş and İ came out as “_” and “0”); without a `font`, such text now logs a warning instead of going wrong silently.
- `setExportDefaults` takes a format — CSV, Excel or PDF — where it set Excel defaults only.
- `theme` accepts your own theme’s name: `theme="acme"` puts `kanuni-datagrid-theme-acme` on the grid, its wrapper and every popup.

### Changed

- Search, text filters, the header-filter search, highlighting and the lookup editors treat İ, I, ı and i as one letter. Lower-casing “İ” produced “i” plus a combining dot, so “istanbul” never matched “İSTANBUL”.
- Search also matches what a cell shows — `valueFormatter` and `dateFormat` output — not only the stored value.
- The CSV separator follows the locale: `;` for Turkish, German, French, Spanish, Italian, Portuguese, Russian and Arabic, where spreadsheet programs expect it; `,` elsewhere.
- The licence console notice names the product that prints it — a page with only a DataGrid said “KanuniLabs PivotGrid” — and prints once per product, not once per grid. With no key at all the console stays quiet; the badge is the notice.
- Types: `theme` is `GridThemeName` (`GridTheme | (string & {})`) — passing a value is unaffected, but code that reads the prop’s type into a `GridTheme` should use `GridThemeName`. `GridDictionary` has eight new licence keys: a partial `messages` object needs nothing, a dictionary typed as the full `GridDictionary` needs them.

### Fixed

- The React Enterprise bundle imported `recharts` at its top, so an app without it could not build. It is loaded when the chart panel opens, and without it the panel says so.
- A valid key could show the licence badge for a moment while it was being checked. Nothing is shown until the check has finished.
- In React, changing `selection.mode` after the first render had no effect.
- The `fitCells` auto-size could run before any row was on screen, or not at all when the data was there at mount; it now runs once the first data cells have rendered. Cells holding markup (badges, chips) are measured as laid out rather than by their text, so they are no longer cut short.
- React screen-reader announcements were always in English; they come from the dictionary.
- With large data (the worker path), a cell edited to a value not seen before could not be found by search, and sorting read outside its cached order.
- React date and lookup editor popups did not receive the theme class.
- A Vite production build stands an empty module in for an uninstalled optional dependency, so a missing `exceljs` or `jspdf` failed with a cryptic error. The loaders now check what they received.

## PivotGrid v1.4.0 — September 22, 2026

`@kanunilabs/pivotgrid-core@1.4.0` · `@kanunilabs/pivotgrid@1.0.1` · `@kanunilabs/pivotgrid-react@1.2.1` · `@kanunilabs/pivotgrid-enterprise@1.1.0` · `@kanunilabs/pivotgrid-react-enterprise@1.2.0` · `@kanunilabs/licensing@1.3.0`

_Licence keys verify on plain-HTTP intranet pages, and the badge says why a key failed._

### Before you upgrade

- A dictionary typed as the full `PivotDictionary` needs the eight new licence keys; a partial `dictionary` needs nothing.

### Added

- Licence keys now verify on pages served over plain `http://`. Browsers only offer `crypto.subtle` on HTTPS and `localhost`; where it is missing, a bundled JavaScript implementation of the same ECDSA P-256 check runs instead — same verdict, the signature is always verified, loaded only on those pages.
- The licence badge says why a key failed — expired, not valid for this domain, revoked, malformed, invalid signature, or “Browser cannot verify the license (HTTPS required)” — in the grid’s language, all twelve; right to left in Arabic. The licence state carries a matching `reasonCode`.

### Changed

- The licence console notice names the product that prints it and prints once per product; with no key at all the console stays quiet.
- `PivotDictionary` has eight new licence keys: a partial `dictionary` needs nothing, a dictionary typed as the full `PivotDictionary` needs them. The Community packages are republished with them and pinned to the same `pivotgrid-core`, so a page never loads two copies.

## DataGrid v1.3.0 — September 19, 2026

`@kanunilabs/datagrid-core@1.2.1` · `@kanunilabs/datagrid@1.2.1` · `@kanunilabs/datagrid-react@1.1.2` · `@kanunilabs/datagrid-enterprise@1.3.0` · `@kanunilabs/datagrid-react-enterprise@1.2.0` · `@kanunilabs/licensing@1.2.0`

_Paste and the fill handle respect cell permissions; paste lands on the focused cell; paid licence keys no longer expire into a watermark._

### Fixed

- Paste and the fill handle wrote into cells the user could not edit. `editable: false` and the `allowUpdating` predicate held for the cell editor and for cut, but Ctrl+V and a fill-handle drag went straight through them. Refused cells are now skipped and the rest of the gesture is written, the same rule cut already followed.
- With no rectangle selected, Ctrl+V pasted into the first row and first column of the grid. A bare arrow key collapses the rectangle, so arrowing down a column and pasting wrote far from where the user was. Paste now lands on the focused cell, or on the cell a context menu was opened over.
- In React, Ctrl+V and Ctrl+X still worked with `editing.range: false`. Cut and paste — keyboard and context menu — now follow the same switch in both renderers.
- The fill handle treated anything `Number()` could convert as a number: two blank cells filled with zeros, two dates filled with epoch-millisecond numbers, `true`/`false` became 1/0. Only real numbers form a series now; everything else repeats as a pattern.
- Community: `pagination: { pageSize: 25 }` without `enabled` left the grid unpaginated. A pagination object now turns paging on unless it says `enabled: false`.
- Community: on touch devices the toolbar, pager and header-filter buttons get 40px touch targets, and the header filter button — revealed only on hover until now, so unreachable without a mouse — is always visible.

### Changed

- Paid Enterprise licence keys are now issued as perpetual keys for the builds released during the paid period: when a subscription ends, the version you already ship keeps working without a watermark. Each Enterprise package checks the key against its own release date.
- `useGridEditing` takes a `range` option, and `paste()` on both renderers takes an optional target cell — the context menu passes the cell it was opened over.

## PivotGrid v1.1.4 — September 19, 2026

`@kanunilabs/pivotgrid-react-enterprise@1.1.4` · `@kanunilabs/pivotgrid-enterprise@1.0.2` · `@kanunilabs/licensing@1.2.0`

_Paid licence keys no longer expire into a watermark._

### Changed

- Paid Enterprise licence keys are now issued as perpetual keys for the builds released during the paid period: when a subscription ends, the version you already ship keeps working without a watermark. Both Enterprise packages pin `@kanunilabs/licensing` 1.2.0, the same copy as the DataGrid Enterprise packages.

## DataGrid v1.2.1 — September 6, 2026

`@kanunilabs/datagrid-enterprise@1.2.1` · `@kanunilabs/datagrid-react-enterprise@1.1.2` · `@kanunilabs/licensing@1.1.2`

_Enterprise packages republished on licensing 1.1.2; the JavaScript licence badge follows fullscreen._

### Fixed

- In the JavaScript Enterprise package the unlicensed badge stayed on the document body while the grid was fullscreen — where the browser does not paint it, so the notice silently vanished. It now mounts inside the fullscreen element and follows the toggle both ways.

### Changed

- Both Enterprise packages pin `@kanunilabs/licensing` 1.1.2, whose LICENSE names all four Enterprise packages. The verifier code is unchanged; the republish keeps a customer who installs both products on one copy of the licensing module, as the 2026-08-26 release promised.

## PivotGrid v1.3.0 — September 6, 2026

`@kanunilabs/pivotgrid-core@1.3.0` · `@kanunilabs/pivotgrid@1.0.0` · `@kanunilabs/pivotgrid-enterprise@1.0.1` · `@kanunilabs/pivotgrid-react@1.2.0` · `@kanunilabs/pivotgrid-react-enterprise@1.1.3` · `@kanunilabs/licensing@1.1.2`

_PivotGrid without React — two new packages — and four engine options that were accepted and ignored now work._

### Before you upgrade

- Column subtotals are on by default, like row totals. Set `showColumnTotals: false` in `layout` to keep the columns you had.
- Exported sheets name their totals in the grid's language instead of English.

### Added

- `@kanunilabs/pivotgrid` is the PivotGrid for plain JavaScript. One `createPivotGrid(element, config)` call builds the grid straight into the DOM — no component tree, no framework peer. It is a second renderer over the same engine and the same stylesheet as the React package: aggregation in the Web Worker, drag-and-drop between areas, filters, sorting, expand and collapse, saved layouts and theming behave identically, and the config mirrors the React props by name. A UMD build (global `KanuniLabsPivotGrid`) makes a script tag a complete install, and a parity ledger checked in CI records, surface by surface, where the two renderers agree.
- `@kanunilabs/pivotgrid-enterprise` brings the paid surface to JavaScript through `createEnterprisePivotGrid()`: the calculated-field designer, the visual prefilter builder, drill-down to source records, an eight-type chart panel and styled Excel and PDF export. It wraps the Community renderer rather than forking it, and the same licence key covers it. The charts are drawn as SVG by the package itself — there is no charting dependency — and the export libraries (`exceljs`, `jspdf`, `jspdf-autotable`) are optional peers, looked up on the page first so a script tag satisfies them.
- `topN` works, and `topNShowOthers` folds the values past the cut into a single Others row or column instead of dropping them.
- The coarse `layout` options — `showGrandTotals`, `showSubTotals`, `showColumnTotals` and `totalsDisplayMode` — are bridged to the flags the engine reads. Until now all four were declared, accepted and read by nothing.
- The three scale-based conditional-formatting rule types — `colorScale`, `dataBar` and `iconSet` — render. Before this release only `expression` rules did.
- Subtotals on the column axis: a group of columns gets a total column beside its children, following `totalsPosition`, and the exporters pick it up because they derive their columns from the same list the screen does.
- In the React renderer: the node context menu is Community now (`PivotNodeContextMenu` is exported from `@kanunilabs/pivotgrid-react`); the field chooser has a Select all button and marks the fields that carry a filter; the field chips are reachable from the keyboard; and a chip being dragged shows a floating clone.

### Changed

- Column subtotals are on by default, like row totals. Set `showColumnTotals: false` for the column set you had before.
- Under `pointer: coarse` the interactive controls get 40px touch targets (the small chip controls through a larger hit area), and `touch-action` is set so a drag does not also scroll the page.
- Exported sheets name their totals in the grid language: the export helpers take the translated Grand Total and Total labels instead of hardcoding English.
- `@kanunilabs/pivotgrid-react-enterprise` is republished on this core; its node context menu delegates to the Community one.
- The two Enterprise packages went out twice the same day: first as 1.0.0 and 1.1.2, then as 1.0.1 and 1.1.3 re-pinned to `@kanunilabs/licensing` 1.1.2, whose LICENSE now names all four Enterprise packages — the JavaScript one included. The verifier itself is unchanged.

### Fixed

- Eight fields in the row area made the data area disappear: the pinned row headers were wider than the viewport and the data cells scrolled underneath them. The row-header block now has a viewport budget, and resizing one of those columns commits the configured width that yields the width you dragged to — previously widening a column could make it narrower.
- A prefilter on a grouped date field compared the wrong encoding and returned the wrong row count. It is evaluated on the encoded column, as every other filter is.
- A row header whose value is 0, an empty string or false exported as blank or as the subtotal caption, because the check was truthiness rather than null.
- Two sweeps over the React renderer closed 22 further verified defects, among them the column Grand Total header ignoring the dictionary.

## PivotGrid v1.2.0 — August 27, 2026

`@kanunilabs/pivotgrid-core@1.2.0` · `@kanunilabs/pivotgrid-react@1.1.0` · `@kanunilabs/pivotgrid-react-enterprise@1.1.1`

_Three configuration props that did nothing now work, and Enterprise export is properly gated._

### Before you upgrade

- `stateStoring` does what it says: `enabled: false` stops saving, `type: 'sessionStorage'` saves there, and `type: 'custom'` calls your `onSave` / `onLoad`. Before, every setting saved to localStorage. Passing no `stateStoring` keeps the old behaviour, so saved layouts are kept.
- The `toolbar` prop is honoured: switches you set in it take effect now.
- On a Community grid the paid export formats (`excel-list`, `excel-pivot`, `pdf`) reject with an error routed to `onError` instead of silently doing nothing, and `exportData` returns a promise.

### Fixed

- `stateStoring` accepted four members and read none of them. `enabled: false` still wrote to localStorage, `type: 'sessionStorage'` still wrote to localStorage, and the `type: 'custom'` pair of `onSave`/`onLoad` — documented as the way to persist to your own backend — was a hook nothing ever called. Only `storageKey` was read, and only as a grid-id fallback. All four work now. Passing no `stateStoring` at all keeps the previous behaviour, so an upgrade does not lose anyone’s saved layout.
- The `toolbar` prop was declared and never destructured, so every switch inside it was inert and the footer always rendered its default set. It is honoured now.
- `onError` never reached the grid. It was pulled out of the props for the error boundary and not passed on, which also meant `PivotControllerOptions.onError` had never fired in React at all.

### Changed

- The three paid export formats — `excel-list`, `excel-pivot` and `pdf` — now refuse on a Community grid instead of quietly doing nothing. The exporter registers itself into a module-level slot when the Enterprise package is imported, and that slot cannot tell which grid is asking, so one Enterprise import anywhere in a bundle used to enable styled export for every grid in it. The controller can tell, so the check lives there. It is an EDITION gate, not a licence gate: an unlicensed Enterprise build still exports, watermarked.
- The refusal is a rejected promise carrying the format and the package that provides it, raised before the `exporting` event and routed to your `onError`. Previously it was a console warning and a resolved promise fired AFTER that event — so a spinner started on it span forever with a clean console. If your application registers its own exporter under one of those format ids, yours still wins.
- `PivotGridRef.exportData` returns a promise rather than `void`, so a refusal can be awaited and caught. Existing callers ignore the return value and are unaffected.

### Added

- Four guides for the paid features, which previously shared one sentence saying they “light up automatically”: calculated fields, the prefilter and node menu, drill-down and charts, and styled export. Each documents the configuration, the API, and where the feature stops.
- `enterpriseEdition` on `PivotControllerOptions`, for applications that build their own shell out of `usePivotController` and the body/footer components rather than using `EnterprisePivotGrid`.

## DataGrid v1.2.0 — August 26, 2026

`@kanunilabs/datagrid@1.2.0` · `@kanunilabs/datagrid-enterprise@1.2.0` · `@kanunilabs/datagrid-core@1.2.0` · `@kanunilabs/datagrid-react@1.1.1` · `@kanunilabs/datagrid-react-enterprise@1.1.1`

_Six config options that were accepted and did nothing now work._

### Fixed

- The JavaScript renderer accepted six configuration members and silently did nothing with them. `tooltip: { enabled: false }` did not turn tooltips off — the one spelling anyone writes to disable a feature. `cellMenu` took only a boolean, so the object form was dropped. `cellClassName`, `headerTooltip` and `renderOverlay` were read and discarded, and `autoColumns` ignored its object form as well as `false`. Every one of them is the worst kind of bug to find: the code looks right, the grid renders, and nothing tells you the line you wrote is inert. They all work now, and the warning table that was supposed to catch this had a hint that pointed the wrong way — that is fixed too.
- A sparkline told screen readers the wrong number. The default label read out an internal drawing coordinate rather than the value: a series closing at 31 was announced as "ending at 4", and the number went DOWN as the line went up. It now reads the last plotted value. Pass `ariaLabel` if you want to say something better.
- Per-instance label overrides for the edit form and the chart panel (`editing.formLabels`, `charting.labels`) were accepted and ignored. They now merge into that grid’s dictionary only — two grids on one page keep their own wording.

### Added

- Three types that the API reference linked to but the package did not export: `CellMenuOptions`, `RowTemplateContext` and `ExportKind`. TypeScript users could see them in the docs and not import them.
- `SparklineGeometry` now carries `lastValue` — the last plotted reading in data units, next to the existing `last` coordinate. If you draw your own sparkline from the geometry, this is the number to put in the label.

## PivotGrid v1.1.0 — August 26, 2026

`@kanunilabs/pivotgrid-core@1.1.0` · `@kanunilabs/pivotgrid-react-enterprise@1.1.0` · `@kanunilabs/pivotgrid-react@1.0.7` · `@kanunilabs/licensing@1.1.1`

_An English grid no longer shows a Turkish spinner._

### Fixed

- Twelve locales shipped, but any key missing from a translation fell through to TURKISH, because Turkish was the merge floor rather than English. The loading text existed only in the Turkish file, so an English grid displayed “Yükleniyor…”. English is the floor now, and roughly 370 previously-untranslated strings were filled in across the ten non-English, non-Turkish locales.
- Eight footer button tooltips were hardcoded English regardless of locale. They go through the dictionary like everything else.

### Added

- The five Enterprise components export their props types (`CalculatedFieldModalProps`, `ChartFactoryProps`, `PivotDrillDownModalProps`, `PivotNodeContextMenuProps`, `PivotPrefilterBuilderProps`). Without them the API reference showed a type name you could read and not import, and the components could not be wrapped in typed application code.
- Every published package now has an API reference, the paid ones included — 507 pages across five surfaces per product, generated from the TypeScript source, plus full-text search across all of it.

### Changed

- PivotGrid Enterprise had been pinned to older Community packages since 1.0.5, so installing both products could give you two copies of the licensing module — and a licence key set through one product was invisible to the other. Everything is republished together on matching versions; one key covers both, as documented.

## DataGrid v1.1.1 — August 25, 2026

_Column resizing works again in the JavaScript renderer._

### Fixed

- Dragging a column edge in the JavaScript renderer moved it by one pixel and then stopped, which looked like resizing was simply dead. The first width change repaints the header, and that rebuilds the grip the drag started on — a captured element removed from the document loses its pointer capture, so the browser stopped delivering the move events. The drag now listens on the window and survives the repaint. React was never affected: it reconciles the header and keeps the same node.
- The standalone JavaScript demo page is gone; `/playground/datagrid/javascript` now lands on the gallery with the JavaScript renderer selected. Every demo there already had the React — JavaScript switch, so the separate page was the same claim made once instead of 82 times — and it had drifted, missing the row-count picker the gallery demos have.

## DataGrid v1.1.0 — August 25, 2026

_Footer totals are free, and a column can bring its own editor._

### Changed

- Footer column totals now work in Community, along with totals-row placement and custom reducers. They were stripped from the config before — while the footer MENU was never gated, so anyone could click a footer cell and pick Sum and get the very total the config had been refused. One capability cannot have two doors with opposite answers; the free side won, because taking the menu away would have removed something people already use. Per-group totals stay Enterprise, since they ride with multi-level grouping.

### Added

- A column can replace its cell editor outright in the JavaScript renderer too: `renderEditor` returns a DOM node, with the same `commit()` / `cancel()` handles the built-in editor uses, so undo/redo, batch mode and validation behave identically. It differs from React in one way, by necessity: React re-invokes the renderer on every validation change, which would destroy a DOM node’s caret and any open dropdown — so the editor is built once and subscribes through `onErrorChange`.

### Fixed

- The React packages catch up with everything since 1.0.2: band borders that were invisible against the header, group-panel chip reordering that was off by one, clipboard copy that wrote the stored code instead of the label you could see, and a chart panel that walked itself off screen in fullscreen.

## DataGrid v1.0.0 — August 25, 2026

_DataGrid without React — the JavaScript renderer ships._

### Added

- @kanunilabs/datagrid on public npm: the same grid built straight into the DOM. One `createGrid(element, config)` call, no component tree, no JSX, no build step required.
- @kanunilabs/datagrid-enterprise from the private registry, with the editing, range work, charts, master-detail, tree data and styled export the React Enterprise package has. The same licence key covers it — nothing to buy again.
- A UMD build in both packages, so a `<script>` tag is a complete install: `KanuniLabsDataGrid.createGrid(...)` with the engine bundled in, no import map and no bundler configuration.
- The config mirrors the React props by name, so a component reads across nearly unchanged. Where a name genuinely differs (`groupPanel` → `grouping: { panel: true }`), the grid says so in the console with the correct spelling instead of ignoring it.
- Every demo in the playground now has a React ⇄ JavaScript switch: the live grid AND the copyable code sample both follow it, so the JavaScript renderer can be evaluated feature by feature rather than on one showcase page.

### Changed

- @kanunilabs/datagrid-core moves to 1.1.0. It gained the helpers both renderers now share — cell display text, anchored-layer placement, sparkline geometry, summary labels and theme presets — so the JavaScript and React renderers resolve one engine rather than each carrying a copy.

## PivotGrid v1.0.6 — August 18, 2026

_npm page and metadata polish — no code changes._

### Changed

- The npm page opens with an animated demo of the component instead of a static screenshot.
- Package metadata now links the public issue tracker at github.com/kanunilabsdev/support — bugs and feature requests no longer require an account on our site.

## DataGrid v1.0.2 — August 18, 2026

_npm page and metadata polish — no code changes._

### Changed

- The npm page opens with an animated demo of the component instead of a static screenshot.
- Package metadata now links the public issue tracker at github.com/kanunilabsdev/support — bugs and feature requests no longer require an account on our site.

## PivotGrid v1.0.5 — August 16, 2026

_The Web Worker setup step is gone._

### Before you upgrade

- The Web Workers are built into the package and start from a `blob:` URL: a worker file you copied into `public/` can go, and an explicit `workerUrl` still wins. Under a Content-Security-Policy without `worker-src blob:`, the grid loads the worker file from the package instead.

### Changed

- Zero-configuration Web Workers: all three workers (aggregation, export, import) are now compiled into the package and started from a Blob URL. There is no file to copy into `public/`, no bundler recipe and nothing to import — Vite, webpack, Next.js and Parcel work as they are. This is the same change DataGrid shipped in 1.0.0.
- Existing setups keep working: an explicit `workerUrl` still wins, so a CDN-hosted or self-served worker is unaffected. Under a Content-Security-Policy that forbids `blob:` workers, the grid falls back to loading the worker file from the package instead of failing.
- `@kanunilabs/pivotgrid-core` grew by 30 KB (173 KB → 203 KB) as the price of embedding three workers.

## DataGrid v1.0.1 — August 16, 2026

_Same-day patch for Node consumers._

### Fixed

- Importing @kanunilabs/datagrid-core in Node (SSR builds, scripts, test runners) kept the process alive after it finished. A module-scope MessageChannel used for cooperative yielding was unref’d before its message handler was attached — and attaching the handler re-refs the port. The order is now correct, so processes exit normally. Browser behaviour is unchanged.

## DataGrid v1.0.0 — August 15, 2026

_DataGrid is here — the second component, covered by the same licence._

### Added

- @kanunilabs/datagrid-core and @kanunilabs/datagrid-react on public npm; @kanunilabs/datagrid-react-enterprise ships from the private registry, like PivotGrid’s.
- One licence key opens both components — an existing PivotGrid key already carries the DataGrid entitlement, with nothing to buy again.
- Community: virtual scrolling to a million rows, Web Worker offload, filter row + header filter popup, grouping with summaries, pinning, selection and keyboard navigation, CSV export, saved views.
- Enterprise: cell and row editing with validation and undo/redo, range selection and fill handle, multi-level grouping, master-detail, tree data, charts, styled Excel/PDF export, import wizard, side bar tool panels, status bar, filter builder.
- Zero-configuration Web Worker: the worker is embedded in the package, so no `workerUrl` and no file copying — the setup step PivotGrid needed is gone.
- Six themes plus a compact density mode, and a chart palette that follows the theme.

## PivotGrid v1.0.4 — July 17, 2026

_Keyboard-first header filters._

### Added

- Filter popup: pressing Backspace while a list item has focus jumps back to the search box and deletes the last character — search, navigate (↑/↓), toggle (Space) and apply (Enter) now work entirely from the keyboard.

## PivotGrid v1.0.3 — July 17, 2026

_npm packaging & documentation polish (covers 1.0.1–1.0.3)._

### Changed

- npm package pages overhauled: product screenshot, feature tour, expanded keywords and contact metadata.
- New bundler guides: Vite worker setup via the `?url` asset import (no manual copying) alongside the Next.js guide.

### Fixed

- Clarified the localhost licensing note: without a key the grid runs watermarked everywhere; localhost only relaxes the domain binding of a valid key.
- API reference README/index URLs now redirect to the API overview instead of returning 404.

## PivotGrid v1.0.0 — July 12, 2026

_General availability — first stable release on npm._

### Added

- @kanunilabs/pivotgrid-core and @kanunilabs/pivotgrid-react published on public npm; the enterprise package ships from the private registry with per-customer credentials.

## PivotGrid v1.0.0-beta.1 — June 25, 2026

_First public beta of the React packages._

### Added

- PivotGrid core engine: Web Worker compute, columnar storage, trie-based grouping.
- React components: BasePivotGrid, field panel, drag & drop, header filters, theming.
- Enterprise: calculated fields, prefilter builder, drill-down, chart integration.
- 12 locales with full RTL support.
- Excel & PDF export (streamed off the main thread).
- Offline license verification and a customer dashboard.

## PivotGrid v0.9.0 — June 20, 2026

_Engine hardening and framework-agnostic core split._

### Changed

- Split the engine into a framework-agnostic core consumed by the React layer.
- Reworked the aggregation pipeline for large datasets (50k+ rows stay responsive).

### Fixed

- Next.js Web Worker resolution via the new `workerUrl` prop.
- Chart panel layout and stable chart sync (no more re-render loops).

---

Install: [PivotGrid](https://www.npmjs.com/package/@kanunilabs/pivotgrid-react) · [DataGrid](https://www.npmjs.com/package/@kanunilabs/datagrid-react) · Docs: <https://www.kanunilabs.com/docs>
