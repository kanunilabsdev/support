# Changelog

Release notes for the KanuniLabs components. Also published at
<https://www.kanunilabs.com/changelog>.

> This file is generated from the source of truth in the product repository.
> Please do not edit it by hand — open an issue instead if something is wrong.

PivotGrid and DataGrid are versioned independently, so entries name the
component they belong to. Entries before August 2026 predate DataGrid and are
PivotGrid releases.

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
