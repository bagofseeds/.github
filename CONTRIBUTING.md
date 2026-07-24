# Contributing to bagofseeds

bagofseeds hosts **[`fiery`](https://bagofseeds.github.io/fiery/)** — a
namespace package of small, single-purpose PyTorch utilities. Each utility is a
"**match**": it installs on its own, depends on as little as possible (`torch`,
other `fiery` matches when useful, well-known packages only when necessary), and
imports under the shared `fiery.` namespace. Matches are re-organized forks of
the `torch-*` packages published under
[`balbasty`](https://github.com/balbasty).

Repositories:

```
fiery              the umbrella / landing page (https://bagofseeds.github.io/fiery/)
fiery-things       the template every new match is generated from
fiery-<name>       one match per repo (fiery.<name>): interpol, bounds, distmap, diffeo, namedtensors, ...
.github            org profile + this guide + the org-wide CLAUDE.md
actions            the reusable GitHub Actions every match's workflows call
```

Each match repo has a **`CLAUDE.md`** describing its role, layout, and any
conventions specific to it. Read it before changing that repo's code. When a
change spans several matches, say so and land the parts as separate PRs.

Keep each match doing its one job. **Prefer deleting code to adding it.**

---

## Issue-based workflow

Work is tracked in issues. Before writing code:

1. **File an issue** describing the change (or claim an existing one). One issue
   = one coherent change.
2. **Break up big work into sub-issues.** A multi-match or multi-part effort
   gets an umbrella issue with a checklist, and each part its own issue that
   references the parent (`part of #NN`). Land the parts as separate PRs; close
   the umbrella when the last one merges. (GitHub sub-issues are used where
   available; otherwise a task list in the umbrella body.)
3. **Triage with a label** (see below) — every issue should carry at least one.
4. **Reference the issue** from your branch and PR, and close it from the PR
   body with `Closes #NN`. Use `part of #NN` when a PR advances but doesn't
   finish an issue.

Say **which match(es)** an issue touches.

### Labels (triage)

| label | for |
|---|---|
| `bug` | incorrect behaviour, crashes, wrong results |
| `enhancement` | new feature or API (e.g. a new name-aware op, a new match) |
| `perf` | efficiency, correctness unaffected |
| `maintainability` | refactors, dedupe, internal cleanup |
| `documentation` | docs, docstrings, CLAUDE.md, this file |
| `compat` | Python- or PyTorch-version compatibility |
| `blocked` | needs a design decision or human input to proceed |
| `good first issue` | small, well-scoped, low-context |

`bug` / `enhancement` / `documentation` / `good first issue` are GitHub
defaults; add the rest once per repo.

---

## Branches, commits, PRs

- **One branch and one PR per task — never bundle.** A branch does exactly ONE
  thing (one issue, one feature, one refactor). If you notice an unrelated bug
  while working, resist fixing it here — file an issue and give it its OWN
  branch. A PR that touches several unrelated concerns is hard to review,
  revert, and bisect; split it. Do this even when a workflow hands you a shared
  branch: rebase each concern onto its own branch.
- **Name the branch for the task** under your author prefix (`claude/…` for
  Claude-authored work, `<user>/…` otherwise), descriptive and referencing the
  issue: `claude/fix-42-view-name-tracking`, `claude/port-namedtensors`.
  (The `fix:`/`feat:`/… prefix belongs on the commit *subject*.) **Never commit
  straight to `main`.**
- **Commit subjects** use a conventional prefix, then a concise imperative
  summary:

  ```
  fix:      a bug / crash / wrong result
  feat:     a new feature or API surface
  refactor: behaviour-preserving restructuring (dedupe, extract, rename)
  perf:     an efficiency change (behaviour unchanged)
  docs:     docs / docstrings / CLAUDE.md / this file
  test:     tests only
  chore:    build, CI, tooling, versioning
  ```

  Explain the *why* in the body, and reference the issue.
- **One PR per branch/task.** Title mirrors the single change; body says what
  changed, why, and **how it was verified** (paste the passing test line and the
  clean `ruff`/`codespell` run), and links the issue. If a repo has a
  `.github/pull_request_template.md`, fill it in.
- **Squash-merge**, then delete the branch.

---

## Build & test gate (must pass before you open a PR)

```sh
pip install .[test]
cd /tmp && python -m pytest <repo>/tests -q     # run from a neutral cwd
ruff check src tests && ruff format --check src tests
codespell src tests
```

- Tests live in `tests/` and run against the **installed** package (native
  namespace + editable installs don't mix — do a regular `pip install`).
- Autograd-bearing ops are checked with `torch.autograd.gradcheck` where
  relevant.
- CI (`.github/workflows/`) runs `ruff` + `codespell`, the pytest **matrix**
  across the supported Python/PyTorch versions, and the docs build, on every PR.

---

## Coding style (Python)

- **PEP 420 native namespace package.** No match ships `fiery/__init__.py`; each
  ships only its `src/fiery/<name>/` subpackage with
  `[tool.setuptools.packages.find] namespaces = true`, so the distributions
  merge into one `fiery` import root.
- **NumPy-style docstrings** and **type annotations** on every public function.
- **`from __future__ import annotations`** at the top of every module.
- **Wide version support.** Matches target a broad range of Python and PyTorch:
  - Keep *runtime-evaluated* code old-compatible (no PEP 695 `type` statements,
    no walrus in code that must run on old Pythons, no `zip(strict=)`, no PEP
    604 `|` / PEP 585 `list[...]` in **values**). Put modern typing in
    **annotations only** — they are lazy strings thanks to
    `from __future__ import annotations` — and build runtime type aliases from
    `typing` generics. Import backported names (`Self`, `Literal`, `Final`, …)
    from `typing_extensions`.
  - When overriding or calling PyTorch functions that may not exist in every
    supported torch version, **guard by feature detection** so the package still
    imports where the op is absent — never register an override for a function
    that isn't there.
- **Ruff** (`line-length = 79`, `select = ["B","E","F","I","W"]`,
  `format.quote-style = "preserve"`) — `ruff check` clean and `ruff format`-ed.
  The standard `ignore` is `["B905","B007","E731"]`.
- **codespell** clean (`[tool.codespell]` in `pyproject.toml`).
- `snake_case` for functions/variables, `_`-prefixed names are internal.
- Match the surrounding code's comment density and idiom. Comments explain
  *why*, not what an obvious line does.

### Review

Correctness-critical changes get a **skeptical diff review** that checks
behaviour against a reference (a brute-force probe, an analytic identity, or
`gradcheck`) before merge. **Correctness beats brevity beats cleverness.**

---

## Packaging, CI & docs

New matches are generated from **`fiery-things`**; study it (and an existing
match such as `fiery-interpol`) before adding a repo.

### pyproject

- **PEP 420 native namespace** (`namespaces = true`, package under `src/`).
- **Dynamic version via `versioningit`** (`dynamic = ["version"]`; setuptools +
  versioningit + wheel backend). The version comes from git tags; a
  `_version.py` is written into the package and git-ignored. `default-tag` keeps
  a tag-less checkout building.
- **Rich `[project]` metadata**: `authors`/`maintainers`, `keywords`, trove
  `classifiers` (incl. `License :: OSI Approved :: MIT License`, the supported
  `Programming Language :: Python :: 3.x`, `Typing :: Typed`) and a
  `[project.urls]` block (Homepage / Documentation / Issues / Repository).
- **Tooling config lives in pyproject**: `ruff`, `codespell`, `coverage`
  (`omit` the generated `_version.py`), and
  `[tool.pytest.ini_options] testpaths = ["tests"]`.

### Workflows

Each repo carries **thin** workflows under `.github/workflows/` that call the
reusable workflows in **`bagofseeds/actions@main`** (one-line callers per
package), so every match inherits shared CI updates without refreshing pinned
SHAs:

- `lint.yaml` — `ruff` + `codespell` (push + PR).
- `test.yaml` / `test-matrix.yaml` — the Python×PyTorch matrix; path-filter on
  `src/**`, `tests/**`, `pyproject.toml`.
- `coverage.yaml` — tests with coverage, uploaded to Codecov.
- `docs.yaml` — build the site and deploy to Pages (`permissions: pages: write,
  id-token: write`). Enable Pages with the **"GitHub Actions" source** once per
  repo (Settings → Pages) or `deploy-pages` fails.
- `publish-*.yaml` — build and publish to (Test)PyPI on release.

### Docs

- **[zensical]** (a mkdocs-material successor) configured in `zensical.toml`,
  with the `mkdocstrings` python handler set to `docstring_style = "numpy"` so
  the API pages render from the same docstrings the code carries. `docs/` holds
  `index.md`, `api.md`, and `requirements.txt`.
- Each match publishes to `https://bagofseeds.github.io/<repo>/`; the umbrella
  **landing page** at `https://bagofseeds.github.io/fiery/` (the `fiery` repo)
  lists every match and each match's nav links back to it.

[zensical]: https://github.com/mkdocs/zensical

---

Thanks for keeping `fiery` small and sharp.
