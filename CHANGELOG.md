# Changelog

Release notes for the KanuniLabs components. Also published at
<https://www.kanunilabs.com/changelog>.

> This file is generated from the source of truth in the product repository.
> Please do not edit it by hand — open an issue instead if something is wrong.

PivotGrid and DataGrid are versioned independently, so entries name the
component they belong to. Entries before August 2026 predate DataGrid and are
PivotGrid releases.

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
- Offline license verification with online revocation and a customer dashboard.

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
