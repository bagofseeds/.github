# Contributing to bagofseeds

bagofseeds publishes two families of PyTorch packages:

- **[`bagof`](https://bagofseeds.github.io/bagof/)** — a namespace package of
  small, single-purpose utilities. Each utility is a "**bag**": it installs on
  its own, depends on as little as possible (other `bagof` packages when
  useful, well-known packages only when necessary), and imports under the shared
  `bagof.` namespace.
- **[`fiery`](https://bagofseeds.github.io/fiery/)** — a namespace package of
  small, single-purpose utilities. Each utility is a "**match**": it installs on
  its own, depends on as little as possible (`torch`, other `fiery` matches when
  useful, well-known packages only when necessary), and imports under the shared
  `fiery.` namespace. The initial matches are re-organized forks of the `torch-*`
  packages published under [`balbasty`](https://github.com/balbasty). Future
  matches will start their life directly under fiery.

Both families share the same workflow, packaging, CI, and docs conventions
(below), but **differ in their supported Python/PyTorch spans and in how they
use annotations** — see *Coding style*. When guidance below differs by family
it says so; otherwise it applies to both.

`fiery` repositories:

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

- **Namespace layout.** `fiery` and `bagof` packages are **PEP 420 native namespace
  packages** — no package ships `<fiery|bagof>/__init__.py`; each ships only its
  `src/<fiery|bagof>/<name>/` subpackage with
  `[tool.setuptools.packages.find] namespaces = true`, so the distributions
  merge into one import root.
- **NumPy-style docstrings** and **type annotations** on every public function.
- **Typing goes through `typing_extensions`** (imported `as tx`, cf.
  `bagof-hints`): don't mix `typing` / `collections.abc` / `tx`, and never
  subscript an abc/builtin generic at runtime (`collections.abc.Sequence[...]`
  is not subscriptable before 3.9).
- **Version span & annotations — this differs by family, do not copy blindly:**
  - **`fiery-*`** targets a wide span (down to old Pythons) and relies on
    **`from __future__ import annotations`** so annotations are lazy strings.
    That is what lets modern typing appear in signatures on old interpreters;
    the price is that only *runtime-evaluated* code (type aliases, defaults)
    must stay old-compatible — no PEP 695 `type` statements, no walrus, no
    `zip(strict=)`, no runtime PEP 604 `|` / PEP 585 `list[...]`. Build runtime
    aliases from `tx.*`.
  - **`bagof-*`** may **use annotations at runtime** (introspection,
    dataclasses, validators). There, `from __future__ import annotations` can be
    **incompatible** and its supported version span is different — follow that
    repo's own policy (its `requires-python` and `CLAUDE.md`); do **not** impose
    the fiery lazy-annotation approach on it.
  - Either way, the concrete supported versions are declared per repo in
    `pyproject.toml` (`requires-python`, trove `classifiers`) and exercised by
    that repo's CI matrix — treat those as the source of truth.
- **Guard PyTorch ops by feature detection.** When overriding or calling torch
  functions that may not exist in every supported torch version, detect them
  first so the package still imports where the op is absent — never register an
  override for a function that isn't there.
- **Lint/format/spelling config lives in each repo's `pyproject.toml`**
  (`[tool.ruff]`, `[tool.ruff.format]`, `[tool.ruff.lint]`, `[tool.codespell]`).
  Don't restate the values here or hard-code them in workflows — read them from
  `pyproject.toml`. The gate is simply: `ruff check` clean, `ruff format`-ed,
  `codespell` clean.
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
