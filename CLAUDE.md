# CLAUDE.md — bagofseeds

Org-wide guidance for coding agents working in the bagofseeds repositories.
Repo-specific details live in each repo's own `CLAUDE.md`; the full human-facing
workflow lives in `CONTRIBUTING.md` (same repo as this file). This file is the
short version an agent should keep in mind.

## The project

bagofseeds publishes two families of PyTorch packages:

- **`bagof-*`** — a namespace package of small single-purpose utilities. Each is a
  "**bag**" in its own `bagof-<name>` repo, installs standalone, and imports
  under the shared `bagof.` namespace (PEP 420, no `bagof/__init__.py`). New
  bags are generated from the **`bagof-things`** template.
- **`fiery`** — a namespace package of small single-purpose utilities. Each is a
  "**match**" in its own `fiery-<name>` repo, installs standalone, and imports
  under the shared `fiery.` namespace (PEP 420, no `fiery/__init__.py`). New
  matches are generated from the **`fiery-things`** template.

Shared CI lives in **`bagofseeds/actions`**; the `fiery` umbrella/landing repo
is **`fiery`**. The two families share the conventions below **but differ in
their supported version spans and annotation strategy** (see conventions).

## Workflow (do not skip steps)

**task → issue → branch → PR → review → merge.**

1. **Issue first.** One issue = one coherent change. Big work gets an umbrella
   issue + sub-issues (`part of #NN`); each part lands as its own PR.
2. **Branch per task**, `claude/<descriptive-name>` referencing the issue.
   Never commit to `main`. One branch does exactly one thing — never bundle
   unrelated changes.
3. **PR per branch.** Body: what changed, why, and how it was verified (paste
   the passing test line + clean `ruff`/`codespell`). `Closes #NN`.
4. **Self-review skeptically** before merge — check correctness against a
   reference, not just that tests are green. Correctness > brevity > cleverness.
5. **Squash-merge**, delete the branch.

## Gate before every PR

```sh
pip install .[test]
cd /tmp && python -m pytest <repo>/tests -q
ruff check src tests && ruff format --check src tests
codespell src tests
```

## Non-negotiable code conventions

- **Version span & annotations differ by family — don't copy blindly.**
  `fiery-*` targets a wide span and uses **`from __future__ import annotations`**
  (annotations are lazy strings), so only *runtime-evaluated* code must stay
  old-compatible: no walrus / PEP 695 `type` / runtime PEP 604 `|` / PEP 585
  `list[...]` / `zip(strict=)`. `bagof-*` may **use annotations at runtime**,
  where `from __future__ import annotations` can be **incompatible**, and its
  version span is different — follow that repo's own `requires-python` and
  `CLAUDE.md`. The per-repo `pyproject.toml` + CI matrix are the source of truth
  for supported versions.
- **Typing via `typing_extensions`** (imported `as tx`, cf. `bagof-hints`): all
  typing — annotations and runtime aliases — goes through `tx.*`; don't mix
  `typing` / `collections.abc` / `tx`, and never subscript an abc/builtin
  generic at runtime.
- **Wide PyTorch.** Feature-detect torch ops before overriding/using them; the
  package must still import where an op is absent.
- **PEP 420 namespace**, `versioningit` dynamic version, NumPy-style
  docstrings, full type annotations. Lint/format/spell config lives in each
  repo's `pyproject.toml` (`[tool.ruff]`, `[tool.codespell]`) — read it there,
  don't restate or hard-code it. Gate: `ruff` + `codespell` clean.
- Thin `.github/workflows/` that call `bagofseeds/actions@main`. zensical docs.

## When in doubt

Prefer deleting code to adding it. Keep each match doing its one job. Read the
repo's `CLAUDE.md` before touching its core, and `CONTRIBUTING.md` for anything
about process, packaging, or CI.
