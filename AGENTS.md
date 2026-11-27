@../phosh-toolkit/agents/core.md

# Domus

The household apps and what they share, in one repository
([decision](docs/decisions/one-repository.md)). Knowledge of the whole:
[docs/index.md](docs/index.md); each part has its own `docs/` and its
own `AGENTS.md`, which apply in addition to this file when you work
there.

| Folder           | What                                                                                     |
| ---------------- | ---------------------------------------------------------------------------------------- |
| `packages/model` | materia, the general model: pure, no domain, no host API                                 |
| `packages/host`  | hospes, the host code two apps need: sync cipher, WebDAV adapter                         |
| `packages/gjs`   | gjs, what the native Linux apps share: Web Crypto subset, libsoup fetch, files, tooling  |
| `apps/mixtura`   | Mixtura (Drink Builder), a workbench for mixed drinks, with a native Linux app           |
| `apps/culina`    | Culina (Cook's Notebook), a daily driver to build dishes, with pairing and taste profile |
| `apps/emptio`    | Emptio (Shopping List), the household's shopping list, with a native Linux app           |
| `apps/domus`     | Household (`@domus/app`): the app at the root of the address, where sync is set up       |
| `scripts`        | checks and the site build that span the parts                                            |

These are web apps (PWA) and libraries, not GTK apps; the GTK rules do
not apply. The exception is an app's native Linux package
(`apps/<app>/packages/linux`,
[decision](docs/decisions/native-linux-gjs.md)), which has its own
`AGENTS.md` with the GTK rules.

## Rules

- **Parts stay apart.** An app imports nothing of another app, and a
  shared package imports nothing of an app; `npm run boundaries` checks
  it. What two apps need moves to a shared package, never in advance
  and never as a copy. The app shell (header, navigation, settings,
  sync display, toasts, banners) is shared
  ([decision](docs/decisions/shared-shell.md)): an app builds none of
  its own.
- **A shared package is changed together with its users.** A change to
  `packages/model` that breaks an app is fixed in the app in the same
  commit; there are no version tags to wait for.
- **Each part keeps its rules.** Read the `AGENTS.md` of the part you
  change: the purity of the model, the sources and licences of data,
  no language model in Culina, and so on.
- **Own local data, offline first.** No server of ours, no account, no
  telemetry, in every app.

## Fixes and reviews

Taken from a study of another agent-built project on 2026-11-20
([reviews](docs/reviews.md#where-these-rules-come-from)).

- **A fix proves its test.** The commit message of a fix names the test
  that pins it, the commit on which that test fails (run it there; do
  not assume), and the neighbouring cases that passed before and after
  (the controls). A test that passes without the fix proves nothing and
  is said to be such or removed.
- **One document per class of defect.** A defect about the order of
  events (a change during a sync cycle, two tabs, a storage that fails
  half way) gets a row in the ordering table of its part, with the
  intended behaviour and the test that pins it; the first is
  [sync orderings](packages/host/docs/sync-orderings.md). An ordering
  that matters and has no test is listed there as not pinned.
- **Say what a test does not prove** next to the claim: a fake storage,
  an injected clock, no real device.
- **Work done by several agents is recorded.** When a change was made
  by more than one agent, [docs/reviews.md](docs/reviews.md) gets an
  entry in the same pull request: who did which part, who checked what
  (an agent does not count as the reviewer of its own part), what the
  checks found, the test counts, and what nobody verified.
- **What a check finds is fixed where it is wrong.** When a browser
  check or a reviewer finds a defect in the app, the app is fixed, never
  the check bent around it, and the entry names the finding.

## Before committing

1. From the root: `npm run check:<part>` for every part you changed
   (`model`, `host`, `gjs`, `mixtura`, `culina`, `emptio`), and `npm run check` (all
   parts, the boundaries and the root files) when you changed
   `packages/`, a root file or more than one part.
2. `docs/` of the part updated in the same commit; a change that
   concerns the whole repository goes into the root `docs/`.
