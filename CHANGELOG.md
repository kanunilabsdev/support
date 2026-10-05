# Changelog

Release notes for the KanuniLabs components. Also published at
<https://www.kanunilabs.com/changelog>.

> This file is generated from the source of truth in the product repository.
> Please do not edit it by hand — open an issue instead if something is wrong.

PivotGrid and DataGrid are versioned independently, so entries name the
component they belong to. Entries before August 2026 predate DataGrid and are
PivotGrid releases.

## DataGrid v1.5.0 — October 5, 2026

`@kanunilabs/datagrid-core@1.5.0` · `@kanunilabs/datagrid@1.4.0` · `@kanunilabs/datagrid-react@1.3.0` · `@kanunilabs/datagrid-enterprise@1.5.0` · `@kanunilabs/datagrid-react-enterprise@1.4.0`

_Tree data from paths, nested arrays or a server; calendar grouping and group selection; relative date filters; many more chart types; and a deferred select-all that reports what it selected._

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
