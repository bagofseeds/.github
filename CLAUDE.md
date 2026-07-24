# CLAUDE.md — bagofseeds

Org-wide guidance for coding agents working in the bagofseeds / `fiery`
repositories. Repo-specific details live in each repo's own `CLAUDE.md`; the
full human-facing workflow lives in `CONTRIBUTING.md` (same repo as this file).
This file is the short version an agent should keep in mind.

## The project

`fiery` is a namespace package of small, single-purpose PyTorch utilities. Each
utility is a "**match**" in its own `fiery-<name>` repo, installs standalone, and
imports under the shared `fiery.` namespace (PEP 420, no `fiery/__init__.py`).
New matches are generated from the **`fiery-things`** template. Shared CI lives
in **`bagofseeds/actions`**; the umbrella/landing repo is **`fiery`**.

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

- **Wide Python (target 3.7+).** Runtime code stays old-compatible; modern
  typing goes in **annotations only** (`from __future__ import annotations`
  makes them lazy). Build runtime type aliases from `typing` generics; import
  backported typing names from `typing_extensions`. No walrus / PEP 695 / runtime
  PEP 604 / `zip(strict=)` in code that must run on old Pythons.
- **Wide PyTorch.** Feature-detect torch ops before overriding/using them; the
  package must still import where an op is absent.
- **PEP 420 namespace**, `versioningit` dynamic version, NumPy-style docstrings,
  full type annotations, `ruff` (line 79, `B,E,F,I,W`, ignore `B905,B007,E731`)
  and `codespell` clean.
- Thin `.github/workflows/` that call `bagofseeds/actions@main`. zensical docs.

## When in doubt

Prefer deleting code to adding it. Keep each match doing its one job. Read the
repo's `CLAUDE.md` before touching its core, and `CONTRIBUTING.md` for anything
about process, packaging, or CI.
