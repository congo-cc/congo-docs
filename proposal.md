# Proposal: CongoCC Documentation — Content and Organization

**Status:** Draft for review
**Date:** 2026-06-12
**Scope:** Deliverable for Phases I–III of `claude_prompts.md`: survey of existing
JavaCC / JavaCC21 / CongoCC documentation, an inventory of what the new
documentation must contain, and alternative organizations for the two target
documents (Reference Manual and User Guide), including proposed `.rst` file
trees and main headings.

---

## 1. Survey of Existing Documentation (Phase I)

### 1.1 Sources examined

| Source | Location | Form | Value for new docs |
|---|---|---|---|
| JavaCC official docs | javacc.github.io/javacc/documentation/ | 6 reference docs: Command Line, Grammar, BNF, API, JJTree, JJDoc | Conceptual baseline (lexical specs, BNF productions, lookahead taxonomy, tree-building concepts). **All syntax shown is now invalid in CongoCC.** |
| JavaCC21 DokuWiki | wiki.parsers.org | 19 wiki pages | The closest thing to a current reference: `new_syntax_summary`, `scan_statement`, `up_to_here`, `contextual_predicates`, `tree_building`, `code_injection`, `include`, `settings`, `new_settings_in_javacc_21`, `deprecated_settings`, `key_differences`, `choice_points`, `first_set`, `convention_over_configuration` |
| CongoCC blog | parsers.org | ~61 articles (2020–2026) in categories: Announcements, Tips & Tricks, JavaCC21, Roadmap, Error Recovery, FreeMarker, Lookahead, Rants | Primary record of feature introductions and evolution; chronological, not consolidated |
| CongoCC BookStack wiki | bookstack.congocc.org | 4 small "books" (newcomer intros, cardinality assertions, playground) | Embryonic; newcomer-level only |
| CongoCC repository | github.com/congo-cc/congo-parser-generator | README, README_RUST, `examples/` (JSON, JSONC, Lua, Python, Java, C#), self-hosted grammar `src/grammars/CongoCC.ccc`, `Main.java` CLI | Ground truth for current syntax, settings, CLI flags, and idiomatic grammar style |
| Getting Started / Migration pages | parsers.org/home/, parsers.org/migrating-to-congocc/ | 2 structured pages | Installation, converter workflow, breaking changes from JavaCC21 |

### 1.2 Key findings

1. **No current reference manual exists anywhere.** The DokuWiki is
   JavaCC21-era (mostly still accurate, but pre-rename and incomplete); the
   BookStack wiki is nearly empty; everything else is blog posts.
2. **CongoCC removed *all* legacy JavaCC syntax** (per the migration page).
   Legacy constructs (`options {...}`, `PARSER_BEGIN/END`, `void Foo() : {} {...}`,
   `LOOKAHEAD(...)`, JJTree decorations, JAVACODE) must appear *only* in
   migration material, never in mainline documentation.
3. **The repository is the only authoritative source** for the current
   keyword set, settings, and defaults. Examples confirmed from
   `CongoCC.ccc` and `examples/json/JSON.ccc`: `UNPARSED` and `SPECIAL_TOKEN`
   are synonyms; `ATTEMPT`/`RECOVER`, `ASSERT`/`ENSURE`, `FAIL`,
   `ACTIVATE_TOKENS`/`DEACTIVATE_TOKENS`, `INJECT`, `INCLUDE`, token-class
   hierarchies via `#Name` annotations, `#if/#endif` preprocessor, and
   fault-tolerant `!` markers are all current syntax.
4. **Blog articles are the only documentation** for several major features:
   token activation, lazy tokens, token chaining, contextual/lexical-state
   tokenization, assertions/ENSURE, FAIL, ATTEMPT/RECOVER, fault-tolerant
   parsing, the preprocessor, EXTRA_TOKENS, MINIMAL_TOKEN, TERMINATING_STRING,
   and the ongoing generated-API evolution.
5. **Four target languages** are now supported (Java, Python, C#, Rust), but
   no per-language documentation exists beyond README_RUST and scattered
   announcements.

---

## 2. Content Inventory (Phase II)

### 2.1 Inherited concepts — still valid, must be rewritten in current syntax

| Concept | Legacy source | Notes for rewrite |
|---|---|---|
| Recursive-descent parsing model, grammar = program | JavaCC Grammar doc | Keep; CongoCC is philosophically the same tool |
| Lexical specification: `TOKEN`, `SKIP`, `MORE`, special/unparsed tokens | JavaCC Grammar doc | `SPECIAL_TOKEN` → prefer `UNPARSED` (synonyms today); brace-free production syntax |
| Lexical states | JavaCC Grammar doc + parsers.org "context-sensitive lexical states" | Extended by token activation; document together |
| Regular expression syntax, private (`<#X>`) regexes | JavaCC Grammar doc | Largely unchanged semantics; full 32-bit Unicode now |
| BNF expansions: sequences, choices `\|`, `( )`, `[ ]`/`?`, `*`, `+` | JavaCC Grammar/BNF docs | Same operators; new postfix style; `=>` and up-to-here additions |
| Lookahead taxonomy: numerical, syntactic, semantic | JavaCC Grammar doc | Reframe entirely around `SCAN`, `=>||`, predicates; nested lookahead now actually works |
| Tree building: per-production nodes, conditional/definite nodes, `#void` | JJTree doc | Now built-in and on by default; JJTree itself is obsolete |
| Token objects carry location info; special tokens chained to regular ones | JavaCC API doc | Now type-safe `TokenType` enums; `Token` implements `Node` by default |

### 2.2 Obsolete — exclude from mainline docs (migration appendix only)

- **Tools:** JJTree (superseded by built-in tree building), JJDoc (no equivalent).
- **File structure:** `options {...}` block, `PARSER_BEGIN(X)...PARSER_END(X)`,
  `TOKEN_MGR_DECLS` (→ `INJECT LEXER`), JAVACODE productions (removed).
- **Constructs:** `LOOKAHEAD(...)` (→ `SCAN` / `=>||`), `jjtThis`, `jjt*` API,
  `XXXConstants` interface (eliminated), `getToken()`-era integer token kinds.
- **Settings removed or now hardwired-on** (from the `deprecated_settings`
  wiki page): `STATIC`, global `LOOKAHEAD`, `CHOICE_AMBIGUITY_CHECK`,
  `OTHER_AMBIGUITY_CHECK`, `FORCE_LA_CHECK`, `BUILD_PARSER`, `BUILD_LEXER`,
  `DEBUG_PARSER`, `DEBUG_LEXER`, `CACHE_TOKENS`, `TOKEN_FACTORY`,
  `USER_DEFINED_TOKEN_MANAGER`, `USER_CHAR_STREAM`, `COMMON_TOKEN_ACTION`
  (→ `TOKEN_HOOK`), `NODE_SCOPE_HOOK` (→ auto-detected hook methods),
  `NODE_EXTENDS` (→ `INJECT BaseNode`), `GRAMMAR_ENCODING` (UTF-8 assumed),
  `UNICODE_INPUT`, `KEEP_LINE_COL`, `ERROR_REPORTING`, `SANITY_CHECK`.
- **JavaCC21 transitional items:** old jar names (`javacc-full.jar`),
  pre-CongoCC package layouts, `LEGACY_GLITCHY_LOOKAHEAD` (document as a
  migration-only compatibility switch).

### 2.3 Current features that the new documentation must cover

| Area | Items | Today's best source |
|---|---|---|
| Invocation | `java -jar congocc.jar grammarfile`; flags `-d`, `-lang` (java/python/csharp/rust), `-n`, `-p sym[,sym]`, `-q`; jar self-update prompt; syntax converter (`convert`) | `Main.java` usage text; Getting Started page |
| Grammar file anatomy | Top-of-file settings (`NAME=value;`), productions, comments, no wrapper blocks | `new_syntax_summary` wiki; example grammars |
| Productions | `Name : expansion ;` form, embedded code actions `{...}`, code prologue (recently fixed semantics) | wiki + "Code Prologue Glitch" article |
| Lexical | `TOKEN`/`SKIP`/`MORE`/`UNPARSED` brace-free productions; lexical states; private regexes; contextual tokens; `IGNORE_CASE`; full Unicode; lazy tokens (`?` suffix semantics per "lazy" article) | wiki + tokenization articles |
| Disambiguation | `SCAN` (numeric, semantic `{...}`, contextual `\...\`/`~\...\`/`/.../`, syntactic, `=>`); up-to-here `=>||`, `=>|+n|`; nested lookahead correctness; choice points & FIRST sets | `scan_statement`, `up_to_here`, `contextual_predicates`, `choice_points`, `first_set` wiki pages; lookahead article series |
| Assertions & failure | `ASSERT {...}` / `ASSERT ~Expansion`, `ENSURE`, `FAIL "msg"` | assertions articles (2021, 2026 ENSURE revision) |
| Tree building | On by default; node-per-production; `#Name`, `#Name(n)`, `#Name(>n)`, `#void`, `#scan`-style conditions; token subclassing (`TOKEN #Delimiter :` and per-alternative `#BooleanLiteral`); `SMART_NODE_CREATION`; base `Node` API; visitor generation | `tree_building` wiki; "just like home-made", "tree-building Dmitry…" articles; JSON example |
| Code injection | `INJECT ClassName : imports/extends/implements { members }`; injection into parser, lexer, base node, token classes; `INJECT LEXER`/`PARSER` | `code_injection` wiki; "fun example of code injection" article |
| Modularity | `INCLUDE "file"`; embedded reusable grammars (e.g., include the Java grammar); C#-style preprocessor `#if SYM / #elif / #else / #endif` with `-p` symbols | `include` wiki; preprocessor article |
| Advanced tokenization | `ACTIVATE_TOKENS` / `DEACTIVATE_TOKENS` (in productions and as setting), token chaining (`TOKEN_CHAINING`), token hooks (`TOKEN_HOOK` methods), `EXTRA_TOKENS`, `MINIMAL_TOKEN`, `TERMINATING_STRING`, whitespace/EOL normalization settings | activation, chaining, token-hooks, tokenization-enhancements articles |
| Fault tolerance (experimental) | `FAULT_TOLERANT`, `FAULT_TOLERANT_DEFAULT`, `!` markers, `ATTEMPT ... RECOVER ...` | fault-tolerant article series; JSON example comments |
| Settings reference | Complete current list w/ defaults — naming (`BASE_NAME`, `PARSER_CLASS`, `LEXER_CLASS`, `PARSER_PACKAGE`, `NODE_PACKAGE`, `BASE_SRC_DIR`), tree (`TREE_BUILDING_ENABLED`, `TREE_BUILDING_DEFAULT`, `TOKENS_ARE_NODES`, `SPECIAL_TOKENS_ARE_NODES`, `NODE_DEFAULT_VOID`, `NODE_PREFIX`, `SMART_NODE_CREATION`), lexical (`DEFAULT_LEXICAL_STATE`, `DEACTIVATE_TOKENS`, `EXTRA_TOKENS`), text handling (`TAB_SIZE`, `PRESERVE_TABS`, `PRESERVE_LINE_ENDINGS`, `ENSURE_FINAL_EOL`, `TERMINATING_STRING`), generation (`MINIMAL_TOKEN`, `TOKEN_CHAINING`, `FREEMARKER_NODES`, `LEGACY_GLITCHY_LOOKAHEAD`), testing (`TEST_PRODUCTION`, `TEST_EXTENSION`), fault tolerance | `settings` wiki page **+ verification against source** (see §2.4) |
| Generated API | Parser class entry points, `Lexer`, `Token`/`TokenType` enum, `Node` interface, node classes, visitors, `ParseException` with grammar-located stack traces, `setBuildTree`/runtime toggles | "API evolution" roadmap article; generated code itself |
| Target languages | Java (≤ current JDK), Python, C#, Rust: per-language invocation, runtime expectations, action-code language, limitations | README, README_RUST, announcement articles |
| Provided grammars | JSON, JSONC, Lua, Python, Java, C# as reusable artifacts and study path (JSON → Lua → Python → Java → C#) | README, `examples/` |

### 2.4 Items requiring verification against the source before writing

These are places where blog/wiki claims may be stale; the writing process
should treat `src/grammars/CongoCC.ccc` and the settings-handling code as
ground truth:

1. The **complete, current settings list with defaults** (the wiki page is
   JavaCC21-era; e.g., `TEST_PRODUCTION`/`TEST_EXTENSION` appear in examples
   but on no wiki page).
2. The **complete keyword/construct set** of the grammar language (e.g.,
   `CONTEXTUAL` tokens appear in `CongoCC.ccc` but have only a blog mention).
3. Current **`convert` subcommand** behavior and supported source dialects.
4. **Environment variables / system properties** honored by the tool (none
   surfaced in the survey; the Reference Manual should state this explicitly
   after checking the source).
5. Per-language **generated API differences** (Python/C#/Rust parity, visitor
   generation, fault-tolerant support coverage).
6. Anything touched by the 2025–2026 "**API evolution**" work (entry-point
   naming, `rootNode()`, token-access methods).

---

## 3. Organization Alternatives (Phase III)

Both documents live in this one Sphinx project, as two top-level `toctree`
sections of `source/index.rst` (which already points at
`docs/reference/reference.rst`). All trees below are rooted at `source/docs/`.

### 3.1 Reference Manual — three candidate organizations

#### Option R1 — Thematic pipeline manual *(recommended)*

Chapters follow the pipeline a grammar author traverses: invoke the tool →
structure the file → define tokens → define syntax → resolve ambiguity →
shape the tree → customize generated code → scale up → handle errors. Mirrors
the proven JavaCC manual shape, modernized.

```
docs/reference/
├── reference.rst          # landing page + toctree for the whole manual
├── overview.rst           # what CongoCC is; processing model; terminology
├── invocation.rst         # CLI flags, converter, exit codes, env/sysprops, build integration
├── grammar-file.rst       # file anatomy, settings syntax, INCLUDE, preprocessor
├── lexical.rst            # token productions, regex syntax, lexical states, Unicode
├── productions.rst        # BNF productions, expansions, operators, embedded actions
├── disambiguation.rst     # choice points, SCAN, up-to-here, predicates, ASSERT/ENSURE/FAIL
├── tree-building.rst      # node annotations, node lifecycle, settings, visitors
├── injection.rst          # INJECT, hooks (TOKEN_HOOK etc.), customization points
├── tokenization-advanced.rst  # activation/deactivation, chaining, lazy, EXTRA_TOKENS, MINIMAL_TOKEN
├── fault-tolerance.rst    # FAULT_TOLERANT, ! markers, ATTEMPT/RECOVER (flagged experimental)
├── generated-api.rst      # parser/lexer/token/node contracts; runtime toggles; exceptions
├── targets/
│   ├── targets.rst        # common matters; choosing -lang
│   ├── java.rst
│   ├── python.rst
│   ├── csharp.rst
│   └── rust.rst
├── settings.rst           # complete alphabetical settings reference (table per setting)
└── appendices/
    ├── meta-grammar.rst   # EBNF of the CongoCC grammar language itself
    ├── legacy.rst         # JavaCC/JJTree/JavaCC21 → CongoCC mapping tables; converter guide
    └── glossary.rst
```

Representative main headings per chapter (abridged):

- **lexical.rst** — Token production kinds (`TOKEN`, `SKIP`, `MORE`,
  `UNPARSED`); Regular expression syntax; Private regexes; String/char
  literals & `IGNORE_CASE`; Lexical states; Contextual tokens; Token class
  annotations (`#Name`); Unicode; How the lexer is generated (NFA notes).
- **disambiguation.rst** — Choice points; Default resolution & FIRST sets;
  `SCAN` forms (numeric / syntactic / semantic / contextual); Up-to-here
  (`=>||`, `=>|+n|`); `=>` shorthand; Nested lookahead semantics; Assertions
  (`ASSERT`, `ENSURE`); `FAIL`; Diagnosing ambiguity warnings.
- **settings.rst** — one entry per setting: type, default, scope
  (grammar/CLI), since-version note, cross-refs to the explaining chapter.

*Pros:* natural learning order **and** serviceable lookup; chapters map
cleanly onto the surveyed material; matches user expectations formed by the
old JavaCC manual.
*Cons:* some constructs straddle chapters (e.g., `ACTIVATE_TOKENS` is both
lexical and syntactic) — needs deliberate cross-referencing and a strong
index (`:ref:` targets per construct).

#### Option R2 — Encyclopedic construct-per-page reference

A short "how to read this manual" overview, then flat alphabetical entries —
one page per construct/keyword (`ASSERT`, `ACTIVATE_TOKENS`, `ATTEMPT`,
`ENSURE`, `FAIL`, `INCLUDE`, `INJECT`, `MORE`, `SCAN`, `SKIP`, `TOKEN`,
`UNPARSED`, `=>||`, `#node-annotations`, …), one page per setting, one page
per CLI flag.

```
docs/reference/
├── reference.rst
├── reading-guide.rst
├── constructs/        # ~25 pages, one per keyword/notation
├── settings/          # ~30 pages, one per setting
├── cli/               # one page per flag + converter
└── api/               # per-language generated API pages
```

*Pros:* best random-access lookup; trivially maintainable (a feature change
touches one file); maps well to future "since version X" annotations.
*Cons:* no narrative — a reader cannot learn the system from it; shared
concepts (choice points, node lifecycle) either duplicate or live in awkward
"concept" pages; ~60+ small files to manage from day one.

#### Option R3 — Language-specification style

A normative spec in the tradition of the JLS: numbered chapters — Notation;
Lexical Structure; Syntactic Structure; Disambiguation Semantics; Tree
Construction Semantics; Embedded Code & Injection; Code-Generation Contract
(per target); Conformance — with formal EBNF excerpts of the meta-grammar
threaded through every section and minimal examples.

*Pros:* maximal precision; future-proof against ambiguity arguments; the
meta-grammar already exists machine-readably (`CongoCC.ccc` is self-hosted),
so formal excerpts can be kept honest.
*Cons:* highest writing cost by far; intimidating to the actual audience
(pragmatic grammar writers); poor fit with the project's informal,
convention-over-configuration culture; duplicates User Guide explanatory load.

### 3.2 User Guide — three candidate organizations

#### Option U1 — Progressive workshop with task-oriented parts *(recommended)*

Part I builds skills in order through real grammars (the repo already defines
the study path JSON → Lua → Python/Java); Parts II–IV are self-contained
how-to chapters; Part V is support material.

```
docs/userguide/
├── userguide.rst            # landing + toctree
├── installation.rst         # jar, Java requirement, editors/IDE notes, building from source
├── tutorial/
│   ├── first-grammar.rst    # minimal grammar; run; inspect generated code & AST
│   ├── json.rst             # walk the bundled JSON grammar; tree annotations in action
│   ├── calculator.rst       # expression grammar: precedence, recursion, choices, SCAN
│   └── beyond.rst           # reading the Lua/Python grammars; where to go next
├── howto/
│   ├── tokens.rst           # designing tokens; tips&tricks distillation; case-insensitivity
│   ├── choices.rst          # resolving choice conflicts idiomatically (=>|| first, SCAN when needed)
│   ├── trees.rst            # shaping useful ASTs; visitors; injected node behavior
│   ├── project-structure.rst# INCLUDE, preprocessor, packages, BASE_SRC_DIR, source control
│   ├── build-integration.rst# Ant/Maven/Gradle/CI; regenerating deterministically
│   ├── testing.rst          # testing grammars & parsers; TEST_PRODUCTION/TEST_EXTENSION
│   ├── context-sensitive.rst# lexical states vs token activation vs contextual tokens — choosing
│   ├── large-inputs.rst     # memory model (whole-file input), gigabyte-scale advice
│   └── fault-tolerance.rst  # IDE-style resilient parsing with ATTEMPT/RECOVER, ! markers
├── explanation/
│   ├── parsing-model.rst    # how recursive descent + scanahead actually work here
│   ├── lexer-internals.rst  # NFA demystified; why no DFA table opts to tune
│   ├── philosophy.rst       # convention over configuration; why legacy syntax died
│   └── freemarker.rst       # template relationship; FREEMARKER_NODES
├── targets/
│   ├── java.rst             # idioms, JDK levels, embedding in apps
│   ├── python.rst
│   ├── csharp.rst
│   └── rust.rst
├── troubleshooting.rst      # common errors & warnings, what they mean, fixes
├── faq.rst
└── migration.rst            # from JavaCC / JavaCC21: converter workflow, checklists
```

*Pros:* serves both newcomers (Part I) and practitioners (Part II+); nearly
every chapter has an identified source article to distill, so writing is
mostly consolidation; troubleshooting/migration give the historical material
a contained home.
*Cons:* "how-to vs explanation" boundary takes editorial discipline; tutorial
chapters must be maintained against tool output drift.

#### Option U2 — Strict Diátaxis quadrants

Top-level split exactly into Tutorials / How-to Guides / Explanation (the
Reference Manual being the fourth quadrant). Same content as U1 but the
quadrant identity is the primary navigation, with rigid page contracts (a
tutorial never explains; a how-to never teaches).

*Pros:* modern documentation best practice; very predictable page purpose.
*Cons:* for a tool with one core workflow, the strict split fragments closely
related material; CongoCC's interesting content is mostly "how-to with
explanation woven in" — strictness costs more than it buys here. U1 keeps the
Diátaxis *spirit* (its Parts correspond to quadrants) without the rigidity.

#### Option U3 — Cookbook / recipes

Organized by end goal: "Parse a data format", "Build a calculator/interpreter",
"Parse a programming language", "Syntax-highlight / IDE tooling (fault
tolerance)", "Reuse the bundled Java grammar", "Port a JavaCC project". Each
recipe is end-to-end with a complete grammar.

*Pros:* immediately practical; great SEO/discoverability; matches how the
blog already teaches.
*Cons:* weak conceptual scaffolding for true beginners; systematic coverage is
hard to guarantee (features that fit no recipe go undocumented); heavy
duplication across recipes.

### 3.3 Comparison summary

| Criterion | R1 thematic | R2 encyclopedic | R3 spec | U1 workshop | U2 Diátaxis | U3 cookbook |
|---|---|---|---|---|---|---|
| Learnability | ●●● | ● | ● | ●●● | ●●● | ●● |
| Lookup speed | ●● | ●●● | ●● | ● | ●● | ●● |
| Writing cost | ●● | ●● | ●●● (highest) | ●● | ●●● | ●● |
| Maintainability | ●● | ●●● | ● | ●● | ●● | ● |
| Fit to surveyed material | ●●● | ●● | ● | ●●● | ●● | ●● |

**Recommendation:** **R1 + U1.** R1 gives the missing "JavaCC-grade" manual
users keep asking for; U1 consolidates six years of articles into a teachable
path. Mitigate R1's lookup weakness with: (a) the alphabetical `settings.rst`,
(b) a construct index page (every keyword → its section anchor), and (c)
Sphinx `genindex` directives on every construct definition. If R2's lookup
style is preferred later, R1 chapters can be split into per-construct pages
mechanically — the reverse (stitching R2 into narrative) is much harder.

---

## 4. Resulting Top-Level Structure

`source/index.rst` becomes the landing page:

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
```

Supporting `conf.py` suggestions (separate, optional change):
- Theme: `sphinx_rtd_theme` or `furo` (alabaster buries deep toctrees).
- Extensions: `sphinx.ext.todo` (mark §2.4 verification gaps inline as
  `.. todo::` during drafting), `sphinx.ext.graphviz` (syntax diagrams,
  lexical-state diagrams), `sphinx_copybutton`.
- Code blocks: start with `.. code-block:: text` for grammar excerpts (no
  Pygments lexer exists for CongoCC); writing a small Pygments lexer for
  `.ccc` is a worthwhile later enhancement.

Proposed writing order (highest reader value first):

1. `userguide/installation` + `tutorial/first-grammar` (unblocks newcomers)
2. `reference/lexical`, `reference/productions`, `reference/disambiguation`
   (the core language, currently scattered across the most sources)
3. `reference/settings` (verified against source — also produces §2.4 answers)
4. `reference/tree-building` + `userguide/howto/trees`
5. Remaining how-tos, targets, fault tolerance, appendices.

---

## 5. Open Questions for Review

1. **Two documents vs. three:** Keep `targets/` duplicated across both docs
   (reference contract vs. usage idioms), or break target-language material
   into a third top-level "Target Language Guide"?
2. **Versioning policy:** CongoCC has rolling releases (a continuously
   updated jar). Should pages carry "current as of <date / jar build>" notes,
   and should we adopt `versionadded`/`versionchanged` directives keyed to
   dates rather than version numbers?
3. **Migration depth:** Is one appendix + one user-guide chapter enough for
   legacy users, or do we want per-chapter "Coming from JavaCC?" callout
   boxes? (Recommendation: appendix + chapter only; callouts age poorly.)
4. **Fault tolerance placement:** Documented as a normal chapter flagged
   *experimental* (proposed), or quarantined into an "Experimental Features"
   part to set expectations harder?
5. **Source-of-truth process:** For the settings reference, do we hand-write
   from the source, or add a small extraction step (parse the settings
   handling in the CongoCC source / query the tool) so the table can be
   regenerated? Hand-written is fine to start; worth deciding before drift.
6. **Upstream coordination:** Should this repo's output eventually replace
   bookstack.congocc.org / wiki.parsers.org content, and is Jonathan Revusky
   reviewing? (Affects tone and how aggressively we may deprecate old pages.)
7. **Rust maturity:** README_RUST is new and Rust support is the youngest —
   document at parity with other targets, or mark provisional?

---

## Appendix A. Source Map (chapter → primary sources)

| Proposed chapter | Primary sources to consolidate |
|---|---|
| reference/lexical | JavaCC Grammar doc (concepts); wiki `new_syntax_summary`; "Tokens, Then and Now"; "All sorts of things… tokenization (Part I)"; "Tokenization Enhancements…" (2026); Unicode announcement; NFA articles |
| reference/productions | wiki `new_syntax_summary`; "Announcing new syntax"; "You say Tomayto…" (2026); "Code Prologue Glitch" (2026); "Ripping out JAVACODE" |
| reference/disambiguation | wiki `scan_statement`, `up_to_here`, `contextual_predicates`, `choice_points`, `first_set`; "The new SCAN construct"; "Lookbehind predicates"; "Nested syntactic lookahead works"; "Revisiting lookahead"; assertions + ENSURE articles; "New feature: FAIL statement" |
| reference/tree-building | JJTree doc (concepts only); wiki `tree_building`; "Tastes just like home-made!"; "Tree building, Dmitry…"; JSON example grammar |
| reference/injection | wiki `code_injection_in_javacc_21`; "A fun example of code injection at work"; "Token hooks revisited" |
| reference/tokenization-advanced | "Activating and De-activating Tokens"; "Token Chaining, JavaCC's Dark Underbelly?"; "ANN: lazy tokens"; "Context-sensitive lexical states"; `TERMINATING_STRING` article; "Some Niggling Whitespace Issues" |
| reference/fault-tolerance | "The Promised Land: Fault-tolerant parsing"; "Fault-tolerant parsing progress"; "ATTEMPT/RECOVER" announcement; JSON example `!` markers |
| reference/settings | wiki `settings`, `new_settings_in_javacc_21`, `deprecated_settings`; **source verification (§2.4)** |
| reference/invocation | `Main.java` usage; Getting Started; "Syntax Converter Available"; preprocessor article (`-p`) |
| reference/targets + userguide/targets | README; README_RUST; JDK/Python version announcements; "JavaCC is not a Java app" |
| reference/appendices/legacy + userguide/migration | parsers.org/migrating-to-congocc/; wiki `key_differences`, `deprecated_settings`; converter articles |
| userguide/tutorial | Getting Started; bundled JSON/Lua grammars; BookStack newcomer books |
| userguide/howto/large-inputs | "Gigabyte is the new Megabyte" |
| userguide/explanation/philosophy | wiki `convention_over_configuration`; "nothingburger"; project history |
| userguide/explanation/freemarker | "Not your Father's FreeMarker!"; `FREEMARKER_NODES` setting docs |
| userguide/testing | `TEST_PRODUCTION`/`TEST_EXTENSION` (needs source verification); repo `tests/` conventions |
