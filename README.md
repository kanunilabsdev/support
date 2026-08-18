# KanuniLabs — issues & release notes

High-performance data components for React: a **PivotGrid** and a **DataGrid**
that stay responsive at a million rows.

- **Docs** — <https://www.kanunilabs.com/docs>
- **Live demos** — <https://www.kanunilabs.com/playground>
- **Pricing** — <https://www.kanunilabs.com/pricing>
- **Changelog** — [CHANGELOG.md](CHANGELOG.md) · also at <https://www.kanunilabs.com/changelog>

### DataGrid

[![KanuniLabs DataGrid for React in action — scrolling, filtering and editing a million rows](https://www.kanunilabs.com/npm/datagrid-react.gif)](https://www.kanunilabs.com/playground/datagrid)

### PivotGrid

[![KanuniLabs PivotGrid for React in action — drag-and-drop pivoting with Web Worker aggregation](https://www.kanunilabs.com/npm/pivotgrid-react.gif)](https://www.kanunilabs.com/playground/pivotgrid)

Both clips are the real components — [drive them yourself in the playground](https://www.kanunilabs.com/playground), no signup, no install.

## What this repository is

A place to **report bugs and request features**, and to follow releases.

**There is no source code here, and there will not be.** KanuniLabs components
are commercial and closed-source; the packages you install from npm are built
from a private repository. If you are looking for the implementation, it is not
public — that is deliberate, not an oversight.

What you get here instead: a public issue tracker you do not need an account on
our site to use, and release notes you can watch.

## Packages

| Package | Registry | Edition |
| --- | --- | --- |
| [`@kanunilabs/pivotgrid-core`](https://www.npmjs.com/package/@kanunilabs/pivotgrid-core) | public npm | Community |
| [`@kanunilabs/pivotgrid-react`](https://www.npmjs.com/package/@kanunilabs/pivotgrid-react) | public npm | Community |
| `@kanunilabs/pivotgrid-react-enterprise` | private registry | Enterprise |
| [`@kanunilabs/datagrid-core`](https://www.npmjs.com/package/@kanunilabs/datagrid-core) | public npm | Community |
| [`@kanunilabs/datagrid-react`](https://www.npmjs.com/package/@kanunilabs/datagrid-react) | public npm | Community |
| `@kanunilabs/datagrid-react-enterprise` | private registry | Enterprise |

The Community packages are free and need no licence key. Enterprise packages
ship from a private registry with per-customer credentials — see
[the registry guide](https://www.kanunilabs.com/docs/datagrid/registry).

## Where to take what

|  | Where |
| --- | --- |
| Bug in a component | [Open an issue](../../issues/new/choose) |
| Feature request | [Open an issue](../../issues/new/choose) |
| Question about the API | [Docs](https://www.kanunilabs.com/docs), then an issue if the docs are unclear |
| **Licence key, invoice, seats, account** | [Your dashboard](https://www.kanunilabs.com/dashboard) or [contact us](https://www.kanunilabs.com/contact) — **not** a public issue |
| **Security vulnerability** | [SECURITY.md](SECURITY.md) — private disclosure, never a public issue |

> [!WARNING]
> Issues here are **public and permanently indexed**. Never paste a licence
> key, a `.npmrc`, registry credentials, an invoice, or customer data into one.
> If you already did: rotate the credential from your dashboard, then edit or
> delete the issue.

See [SUPPORT.md](SUPPORT.md) for what response to expect on which plan.

## Reporting a good bug

The fastest fixes come from reports that let us reproduce the problem without
guessing. Please include:

- the package **name and version** (`@kanunilabs/datagrid-react@1.0.1`, not "latest");
- React version, bundler (Vite / webpack / Next.js), and browser;
- what you expected versus what happened;
- a **minimal reproduction** — a StackBlitz or CodeSandbox beats a description.
  Community packages install from public npm, so a reproduction needs no licence.

Enterprise-only behaviour cannot be reproduced in a public sandbox. Describe the
configuration instead, and we will reproduce it here.

---

© KanuniLabs. The components are commercial software; see
[Terms](https://www.kanunilabs.com/terms).
