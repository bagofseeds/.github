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


## Documentation

This section records the repository owner's instructions for writing and
maintaining documentation in `bagof-*` and `fiery-*` packages: the README, the pages under
`docs/`, and every public docstring under `src/`. Follow it whenever you write
or edit documentation here.


### Docstring mechanics (mkdocstrings / numpydoc)

- Admonitions (`!!! note`, `!!! example`, `!!! warning`, ...) always go
  **before** the `Parameters` / `Returns` / `Raises` / `Yields` sections of a
  docstring. Placed after them, mkdocstrings swallows the admonition into the
  generated table.
- Cross-references spell out the full dotted path only when it is needed. The
  term in the left bracket is always in backticks, for code formatting.
  - Short form, with empty right brackets: [`Function`][] or
    [`issubhint`][]. Use it when the symbol is defined in, or imported into,
    the module where the docstring lives. The relative and scoped
    cross-reference resolution (enabled in `zensical.toml`) finds it.
  - Long form, with the displayed name on the left and the full path on the
    right: [`Protocol`][typing.Protocol] or
    [`issubhint`][bagof.dispatchers.core.issubhint]. Use it only when the
    symbol is not in scope, and for the standard library.

### Documentation writing style

Write documentation as polished technical prose intended to be read by humans.

Clarity and natural English take priority over compactness. Be concise by removing information that does not help the reader, not by compressing sentences, omitting useful words, or avoiding repetition. A slightly longer sentence that reads naturally is preferable to a shorter sentence with awkward grammar or excessive information density.

#### Prose

Write complete, well-formed sentences with a natural rhythm. Sentences should have clear subjects and verbs. Repeat the name of an object, class, method, or concept when doing so makes the sentence easier to follow. Do not treat lexical repetition as something that must be eliminated.

Prefer explicit references over compressed or ambiguous ones. In particular, avoid strings of pronouns, demonstratives, and elliptical references such as “this”, “that”, “it”, “these”, “the former”, or “the latter” when naming the relevant concept would make the sentence clearer.

Do not compress multiple relationships into dense noun phrases merely to save words. Introduce concepts in a logical order and give the reader enough grammatical structure to understand the relationship between them without having to unpack the sentence.

Technical terminology is appropriate when it names a precise concept. Do not use jargon merely because it sounds concise or technical. Prefer ordinary English for the surrounding explanation.

Documentation may use passive constructions when they are natural and when the actor is irrelevant. Do not systematically prefer either active or passive voice. Avoid conversational second-person language unless the documentation genuinely addresses an action performed by the reader.

#### Flow

Documentation should read as continuous prose, not as a collection of compressed facts.

Each sentence should follow naturally from the previous one. Make relationships between ideas explicit when they matter: cause, contrast, sequence, qualification, and consequence should not be left for the reader to reconstruct.

Paragraphs should have a coherent purpose. Do not create a new paragraph for every sentence, but do not build large paragraphs that combine unrelated ideas.

Vary sentence structure naturally. Short sentences are useful for simple statements. Longer sentences are appropriate when they express a relationship that is clearer when kept together. Do not force every sentence into the same terse declarative pattern.

Read the prose mentally as ordinary English. If a sentence would sound awkward when spoken by a careful technical writer, rewrite it.

#### Concision

Aim for high information density without compressed grammar.

Remove:

* redundant explanations;
* repeated conclusions;
* obvious restatements of signatures or type annotations;
* generic introductory sentences that add no information;
* unnecessary qualifiers and filler;
* implementation details that are irrelevant to the documented interface.

Do not remove:

* articles, prepositions, conjunctions, or other words needed for natural grammar;
* a repeated noun when replacing it with a pronoun would reduce clarity;
* a short explanatory clause that makes the logic easier to understand;
* transitions that genuinely help the prose flow.

Do not mistake terseness for precision.

#### Punctuation and formatting

Use punctuation conventionally and unobtrusively.

Use em dashes sparingly. They are appropriate for an occasional genuine interruption or parenthetical remark, not as a default way of joining clauses. Prefer commas, parentheses, colons, semicolons, or separate sentences when those are more natural.

Likewise, do not overuse:

* colons as a substitute for ordinary prose;
* parenthetical remarks;
* sentence fragments;
* slash constructions;
* scare quotes;
* bold text;
* “Note:”, “Important:”, or similar callouts.

Use lists when the content is genuinely enumerable or when a list materially improves referenceability. Do not turn prose into bullet points simply because the information can technically be enumerated.

#### Structure and explanation

Write for a technically competent reader who does not yet know this particular codebase.

Explain what an abstraction represents before discussing incidental implementation details. When documenting behaviour, describe the normal conceptual model first, then exceptions or special cases.

Do not merely translate code into English. Documentation should explain the meaning and purpose of the interface: what an object represents, what an operation does, what assumptions it makes, and what distinctions matter to someone using or maintaining it.

Avoid unnecessary tutorial language. Do not tell the reader to “simply”, “just”, or “remember to” do things. State the relevant behaviour directly.

#### Existing documentation

Do not imitate awkward prose merely because it already exists in the repository. Existing documentation is a source of factual information, terminology, and established naming conventions, but not automatically a style reference.

When extending or editing existing documentation, preserve its technical meaning and established terminology while improving unclear, compressed, repetitive, or unnatural prose. New documentation should follow the rules in this instruction even when nearby documentation does not.

#### Revision test

Before returning documentation, edit it once specifically for prose quality.

For each sentence, ask:

1. Does it have a clear grammatical structure?
2. Is the subject obvious without requiring the reader to decode a pronoun or implicit reference?
3. Has anything been compressed merely to avoid repeating a word?
4. Could unnecessary jargon be replaced by ordinary English without losing precision?
5. Does the sentence connect naturally to the sentences around it?
6. Is every technical detail here useful to the intended reader?
7. Would a competent human technical writer plausibly write the sentence this way?
8. Is the punctuation doing ordinary grammatical work rather than creating an artificial sense of compactness?

If concision and graceful English conflict, first remove unnecessary content. Do not sacrifice natural grammar to make the remaining content shorter.

## Checks

`tests/test_docstrings.py` runs every `pycon` block in the docs and in public
docstrings, on Python 3.8 and on the current interpreter, so every example
must be 3.8-safe: no `Annotated`, no `list[int]` or `X | Y` spelling, nothing
that only a newer Python accepts. Code that cannot run on 3.8 belongs in a
plain ` ```python ` block instead, which is not executed.

Also run `codespell` and `ruff check` over any files you touch.
