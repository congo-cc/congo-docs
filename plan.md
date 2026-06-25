# Plan: CongoCC Documentation — Build Plan for Review

**Status:** Draft for review (supersedes the recommendations in `proposal.md`;
the survey and content inventory in `proposal.md` §1–§2 still stand)
**Date:** 2026-06-24
**Prerequisite to:** writing any `.rst` content

This plan folds in Rich's answers to the seven questions and the fact that
CongoCC source changed materially in the last two weeks. Nothing here is
written yet — this is the plan to approve before generation begins.

---

## 1. Decisions Locked (from the seven answers)

| # | Question | Decision |
|---|---|---|
| 1 | 2 docs vs 3 | **Three** top-level documents: Reference Manual, User Guide, **Target Language Guide** (new). One grammar → four targets; the differences (injected-code language, runtime, build, generated `FIXME.md`/`inject.rs` for Rust) centralize here. |
| 2 | Versioning | Sphinx `versionadded` / `versionchanged`, argument = **ISO date** (e.g. `2026-06-19`). Label text customized so it reads "Added/Changed" rather than "New in version". |
| 3 | Migration | One **appendix** in the Reference Manual (mapping tables + converter workflow) + one **chapter** in the User Guide. No per-chapter callout boxes. |
| 4 | Fault tolerance | Normal chapter, **flagged experimental** (admonition at top). Not quarantined. |
| 5 | Settings source of truth | **Hand-write** the settings reference now; **build a Java extraction tool** (committed to this repo) for future regeneration/verification. |
| 6 | Review & publish | **Jonathan Revusky reviews** before publication; target host is **Read The Docs**. Drives theme, `.readthedocs.yaml`, and conservative deprecation tone toward existing wiki pages. |
| 7 | Rust | **At full parity** with Java/Python/C#. `examples/rust/Rust.ccc` is a complete Rust-language grammar; `serde` support landed 2026-06-08. |

---

## 2. Revised Top-Level Structure

`source/index.rst` carries three `toctree` captions:

```rst
CongoCC Documentation
=====================

.. toctree::
   :maxdepth: 2
   :caption: Reference Manual
   docs/reference/reference

.. toctree::
   :maxdepth: 2
   :caption: User Guide
   docs/userguide/userguide

.. toctree::
   :maxdepth: 2
   :caption: Target Language Guide
   docs/targets/targets
```

The change from `proposal.md`: the `targets/` subtrees that were duplicated
inside the Reference Manual and User Guide are **extracted into the new
Target Language Guide**. The Reference Manual keeps a single *language-neutral*
generated-API contract chapter; everything language-specific moves to the
new guide.

### 2.1 Reference Manual — `source/docs/reference/`

```
reference.rst            # landing + toctree
overview.rst             # what CongoCC is; processing model; terminology
invocation.rst           # CLI flags (-d -lang -n -p -q), converter, exit codes, env/sysprops
grammar-file.rst         # file anatomy, settings syntax, INCLUDE, preprocessor (#if/#define/#undef)
lexical.rst              # TOKEN/SKIP/MORE/UNPARSED, regex syntax, lexical states, Unicode,
                         #   string-literal auto-declaration (changed 2026-06), REQUIRE_TOKEN_DECLARATION
productions.rst          # productions, expansions, operators, embedded actions, code prologue
disambiguation.rst       # choice points, SCAN, up-to-here, predicates, ASSERT/ENSURE/FAIL
tree-building.rst        # node annotations, lifecycle, settings, visitors, language-neutral Node model
injection.rst            # INJECT (language-neutral mechanics), hooks (TOKEN_HOOK etc.)
tokenization-advanced.rst# ACTIVATE/DEACTIVATE_TOKENS, chaining, lazy, EXTRA_TOKENS, MINIMAL_TOKEN
fault-tolerance.rst      # experimental admonition; FAULT_TOLERANT, ! markers, ATTEMPT/RECOVER
generated-api.rst        # LANGUAGE-NEUTRAL contract: Parser/Lexer/Token/TokenType/Node/visitor concepts
settings.rst             # complete alphabetical settings reference (hand-written, source-verified)
appendices/
  meta-grammar.rst       # EBNF of the CongoCC grammar language (from self-hosted CongoCC.ccc)
  legacy.rst             # JavaCC/JJTree/JavaCC21 -> CongoCC mapping tables + converter workflow
  glossary.rst
```

### 2.2 User Guide — `source/docs/userguide/`

```
userguide.rst            # landing + toctree
installation.rst         # jar, Java requirement, editors/IDE, building from source (ant test)
tutorial/
  first-grammar.rst      # minimal grammar; run; inspect generated code + AST
  json.rst               # walk bundled JSON grammar; tree annotations in action
  calculator.rst         # expression grammar: precedence, recursion, choices, SCAN
  beyond.rst             # reading Lua/Python grammars; the JSON->Lua->Python->Java->C# path
howto/
  tokens.rst             # designing tokens; case-insensitivity; distilled tips & tricks
  choices.rst            # resolving choice conflicts idiomatically (=>|| first, SCAN when needed)
  trees.rst              # shaping useful ASTs; visitors; injected node behavior
  project-structure.rst  # INCLUDE, preprocessor, packages, BASE_SRC_DIR, source control
  build-integration.rst  # general workflow: regenerate deterministically, CI (language specifics -> Target Guide)
  testing.rst            # testing grammars/parsers; TEST_PRODUCTION/TEST_EXTENSION
  context-sensitive.rst  # lexical states vs token activation vs contextual tokens — choosing
  large-inputs.rst       # whole-file memory model; gigabyte-scale advice
  fault-tolerance.rst    # using ATTEMPT/RECOVER + ! markers for IDE-style resilience
explanation/
  parsing-model.rst      # recursive descent + scanahead, as actually implemented
  lexer-internals.rst    # NFA demystified; why there's no DFA table to tune
  philosophy.rst         # convention over configuration; why legacy syntax was removed
  freemarker.rst         # template relationship; FREEMARKER_NODES; CTL/template engine notes
troubleshooting.rst      # common errors & warnings, meanings, fixes
faq.rst
migration.rst            # from JavaCC / JavaCC21: converter workflow + checklist
```

### 2.3 Target Language Guide — `source/docs/targets/` (NEW)

```
targets.rst              # landing + toctree
overview.rst             # one grammar, four targets; how -lang works; what actually differs
injected-code.rst        # cross-language INJECT: action-code language per target;
                         #   Rust's generated FIXME.md + inject.rs workflow; Python/C# equivalents
java.rst                 # idioms, embedding, runtime; (JDK_TARGET removed 2026-06 — note current behavior)
python.rst               # runtime expectations, packaging, idioms
csharp.rst               # runtime, preprocessor define/undef rules (changed 2026-06), idioms
rust.rst                 # parity; modular grammar (RustLexer.ccc + RustIdentifierDef.ccc);
                         #   FIXME.md/inject.rs handwritten-code workflow; serde feature
build-and-runtime.rst    # per-language build + runtime-library matrix (quick comparison table)
```

---

## 3. Cross-Cutting Conventions

### 3.1 Versioning directives (answer #2)
- Use `.. versionadded:: 2026-06-19` / `.. versionchanged:: 2026-06-19` /
  `.. deprecated:: <date>` at each construct/setting that changed.
- Override the rendered prefix (default "New in version X") via a Sphinx
  locale override or `locale_dirs` so it reads e.g. **"Added 2026-06-19"** /
  **"Changed 2026-06-19"** — avoids the misleading word "version" for a tool
  with no version numbers.
- A short "How currency is marked" note goes in each landing page.

### 3.2 Settings source of truth (answer #5)
- **Now:** hand-write `reference/settings.rst` from the CongoCC source, not
  the JavaCC21 wiki (which is already wrong — see §4).
- **Tooling deliverable:** a small Java program (proposed
  `tools/settings-extractor/` in this repo, or a `congocc` sandbox utility)
  that reads the settings definitions out of the CongoCC source / a built jar
  and emits an `.rst` table (or `.csv` the docs include). Lets the table be
  regenerated and diffed against the prose. Built once the prose stabilizes.

### 3.3 Sphinx / Read The Docs setup (answer #6)
- Theme: `sphinx_rtd_theme` (canonical for RTD; handles three deep toctrees
  far better than the current `alabaster`).
- Add `.readthedocs.yaml` (build OS, Python version, Sphinx config path) and a
  docs requirements file consistent with the existing `uv`/`pyproject.toml`
  setup.
- `conf.py` extensions: `sphinx.ext.todo` (track §4 verification gaps inline),
  `sphinx.ext.graphviz` (lexical-state / syntax diagrams), `sphinx_copybutton`,
  `sphinx_design` (the experimental-feature admonitions, comparison cards).
- Grammar snippets use `.. code-block:: text` initially; a small **Pygments
  lexer for `.ccc`** is a worthwhile later enhancement (own task, not blocking).

### 3.4 Construct index / lookup mitigation
Because the Reference Manual is thematic (R1), add: an alphabetical
construct-index page (keyword → section anchor), `:ref:` targets on every
construct and setting definition, and rely on Sphinx `genindex`.

---

## 4. Verification Pass Against Current Source (Milestone 0)

Rich's note that source changed in the last two weeks is now concrete. A
verification pass precedes prose for the affected chapters. Confirmed
already-stale or new items the wiki/blog do **not** reflect:

| Item | Change (commit date) | Affects |
|---|---|---|
| `REQUIRE_TOKEN_DECLARATION` | new setting (2026-06-10/11) | settings, lexical |
| `JDK_TARGET` / `jdkTarget` | **removed** (2026-06-08) | settings, targets/java |
| `DEFAULT_LEXICAL_STATE` | semantics reworked (2026-06-08…12) | settings, lexical |
| Implicit string-literal token auto-labeling | new behavior (2026-06-12) | lexical, tree-building |
| String-literal regexp logic | reworked (2026-06-17) | lexical |
| C# preprocessor `#define/#undef` | restricted after first token (2026-06-14) | grammar-file, targets/csharp |
| Rust `serde` feature | completed (2026-06-08) | targets/rust |
| CTL `#loop while/until`, `obj::new(...)` | template-engine internals (2026-06-18/19) | explanation/freemarker (mention only) |

Carried forward from `proposal.md` §2.4 (still to verify from source/jar):
complete current settings list + defaults, full keyword/construct set
(incl. `CONTEXTUAL` tokens), `convert` subcommand behavior, env vars /
system properties, per-language generated-API differences, and the 2025–26
generated-API evolution (entry points, root-node accessor, token access).

**Method:** read `src/grammars/CongoCC.ccc` (self-hosted grammar = ground
truth for keywords/syntax) and the settings-handling code; where behavior is
unclear, build the jar and run it against the bundled examples. This same
reading produces the inputs for the Java extraction tool (§3.2).

---

## 5. Writing Order (Milestones)

0. **Verify against source** (§4) + stand up Sphinx infra (theme,
   `.readthedocs.yaml`, conf.py extensions, three-toctree `index.rst`,
   versioning-label override). *No prose depends on stale data.*
1. **Newcomer on-ramp:** `userguide/installation` + `tutorial/first-grammar`.
2. **Core language (highest value, most scattered today):**
   `reference/lexical`, `reference/productions`, `reference/disambiguation`.
3. **`reference/settings`** (source-verified) → then build the **Java
   extraction tool** to lock it down.
4. **Tree building:** `reference/tree-building` + `userguide/howto/trees`.
5. **Target Language Guide:** `overview`, `injected-code`, then
   `java`/`python`/`csharp`/`rust`.
6. **Remaining how-tos, explanation, fault tolerance, migration, appendices.**
7. **Polish:** construct index, cross-refs, optional `.ccc` Pygments lexer,
   diagrams.

Each milestone is independently reviewable by Revusky.

---

## 6. What I Need to Proceed

1. **Approve the three-document structure and file trees** (§2), or redline.
2. **Confirm `sphinx_rtd_theme` + `.readthedocs.yaml`** approach (§3.3), or
   name a preferred theme (e.g. `furo`).
3. **Settings extraction tool placement** — OK to add `tools/` to *this*
   (congocc-docs) repo (§3.2)? Or should it live in the main CongoCC repo?
4. **A current jar (or OK for me to build one)** for source verification and
   for testing example output during writing (§4). Is `ant test` against a
   local clone the intended path, or is there a published current jar?
5. **Green light to start at Milestone 0** once 1–4 are settled. I will *not*
   write `.rst` content before then.

---

## Appendix. Decisions That Revise `proposal.md`

- `proposal.md` §3 recommended R1 + U1 for **two** documents. This plan keeps
  R1 (thematic reference) and U1 (workshop user guide) but **adds the third
  Target Language Guide** and moves all per-language pages there.
- `proposal.md` listed six bundled grammars (JSON, JSONC, Lua, Python, Java,
  C#). Actual `examples/` also contains `arithmetic`, `cics`, `php`,
  `preprocessor`, `sqlexpr`, and `congo-templates` — the User Guide tutorial
  and "study path" should reference the real, current set.
- `proposal.md` §2.4 verification list is **expanded** by §4 above with the
  concrete last-two-weeks changes.
