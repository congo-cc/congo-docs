# Source Verification Notes (Milestone 0)

**Verified against:** `/home/rich/git/congo-rust` @ `prod` (commit `36d2c8b`,
2026-06-21); jar built 2026-06-23. Toolchain: JDK 21, Ant 1.10.15.
**Purpose:** Ground-truth inputs for the settings/lexical/disambiguation/
fault-tolerance chapters and the future Java settings-extraction tool. These
are working notes, not published prose.

Primary sources read:
- `src/java/org/congocc/app/AppSettings.java` (settings registry — authoritative)
- `src/grammars/CongoCC.ccc` (self-hosted grammar — authoritative for keywords)
- `src/java/org/congocc/app/Main.java` (CLI) + live `java -jar congocc.jar`

---

## 1. Settings registry (authoritative, from AppSettings.java)

Settings are type-classified in three string constants (lines 49–62). Anything
not listed triggers a live warning: *"The option X is not recognized and will
be ignored."* (confirmed by running the jar).

### 1.1 Boolean settings (+ verified defaults)

| Setting | Default | Notes |
|---|---|---|
| `TREE_BUILDING_ENABLED` | **true** | master switch; if false, tree settings warn as meaningless |
| `TREE_BUILDING_DEFAULT` | **true** | tree built unless toggled at runtime |
| `TOKENS_ARE_NODES` | **true** | tokens are terminal nodes |
| `UNPARSED_TOKENS_ARE_NODES` | **false** | requires `TOKENS_ARE_NODES`; falls back to `SPECIAL_TOKENS_ARE_NODES` |
| `SPECIAL_TOKENS_ARE_NODES` | (legacy synonym) | read only as fallback for the above |
| `SMART_NODE_CREATION` | **true** | node created when >1 node on stack |
| `NODE_DEFAULT_VOID` | **false** | |
| `NODE_USES_PARSER` | **false** | |
| `LEXER_USES_PARSER` | **false** | |
| `TOKEN_MANAGER_USES_PARSER` | false | in boolean list; legacy-ish alias of lexer/parser coupling |
| `FAULT_TOLERANT` | **false** | experimental |
| `FAULT_TOLERANT_DEFAULT` | **true** | only relevant when FAULT_TOLERANT |
| `PRESERVE_TABS` | **true** if no TAB_SIZE/TABS_TO_SPACES set, else `tabSize==0` | |
| `PRESERVE_LINE_ENDINGS` | **false** | |
| `ENSURE_FINAL_EOL` | **false** | if true, terminating string defaults to "\n" |
| `JAVA_UNICODE_ESCAPE` | **false** | |
| `IGNORE_CASE` | **false** | global; also a per-token modifier |
| `C_CONTINUATION_LINE` | **false** | implies `USES_PREPROCESSOR` |
| `USES_PREPROCESSOR` | **false** (true if C_CONTINUATION_LINE) | |
| `USE_CHECKED_EXCEPTION` | **false** | ParseException checked vs unchecked |
| `LEGACY_GLITCHY_LOOKAHEAD` | **false** | migration compatibility switch |
| `TOKEN_CHAINING` | **false** (auto-true if grammar calls `preInsert`) | |
| `ASSERT_APPLIES_IN_LOOKAHEAD` | **false** | whether ASSERT fires during lookahead |
| `REQUIRE_TOKEN_DECLARATION` | **false** | **NEW (2026-06-10/11)** — disallows implicit string-literal tokens |
| `X_JTB_PARSE_TREE` | **false** | experimental (X_ = hidden/advanced) |
| `X_SYNTHETIC_NODES_ENABLED` | **false** | experimental |

### 1.2 String settings

`BASE_NAME`, `PARSER_PACKAGE`, `PARSER_CLASS`, `LEXER_CLASS`, `BASE_SRC_DIR`,
`BASE_NODE_CLASS`, `BASE_TOKEN_CLASS`, `NODE_PREFIX`, `NODE_CLASS` *(see dead
note)*, `NODE_PACKAGE`, `DEFAULT_LEXICAL_STATE`, `OUTPUT_DIRECTORY`,
`DEACTIVATE_TOKENS`, `EXTRA_TOKENS`, `ROOT_API_PACKAGE`, `COPYRIGHT_BLURB`,
`TERMINATING_STRING`, `TEST_PRODUCTION`, `TEST_EXTENSION`.

Defaults / derivations verified:
- `PARSER_CLASS` → `<BaseName>Parser`; `LEXER_CLASS` → `<BaseName>Lexer`;
  `BASE_NAME` derives from the grammar filename if unset.
- `PARSER_PACKAGE` → lowercase parser class name if unset.
- `NODE_PACKAGE` → `<PARSER_PACKAGE>.ast` if unset.
- `BASE_NODE_CLASS` → `BaseNode` (or `<BaseName>Node` when `ROOT_API_PACKAGE` set).
- `BASE_TOKEN_CLASS` → `Token` (or `<BaseName>Token` when `ROOT_API_PACKAGE` set).
- `BASE_SRC_DIR` and `OUTPUT_DIRECTORY` are **synonyms**; default `.`.
- `DEFAULT_LEXICAL_STATE` default `DEFAULT` (semantics reworked 2026-06-08…12).
- `TERMINATING_STRING` default `""` (or `"\n"` when `ENSURE_FINAL_EOL`).
- `NODE_PREFIX` default `""`.
- `EXTRA_TOKENS`: comma-list `NAME` or `NAME#ClassName`; default class `<NAME>Token`.
- `DEACTIVATE_TOKENS`: comma/space list of token names off by default.

### 1.3 Integer settings

`TAB_SIZE`, `TABS_TO_SPACES` — **synonyms** (TAB_SIZE falls back to
TABS_TO_SPACES); default effective value `1`.

### 1.4 Deprecated / dead / synonym map (document, don't promote)

- **`JDK_TARGET`** — *deprecated no-op*. Not in any type list; specially handled
  to warn: *"Option JDK_TARGET is deprecated and currently has no effect. (It
  never did!)"* (AppSettings:231–233). Plan §4 said "removed" → correct to
  "deprecated/no-op". (Quick CLI test didn't surface the warning text; confirm
  exact emission path when writing the chapter.)
- **`NODE_CLASS`** — listed in `stringSettings` (twice) but **no getter and no
  use** anywhere in `src/java`. Effectively dead/legacy; `BASE_NODE_CLASS` is
  the live setting. Document as legacy/ignored.
- **`SPECIAL_TOKENS_ARE_NODES`** — legacy synonym, only read as a fallback for
  `UNPARSED_TOKENS_ARE_NODES`.
- **`TABS_TO_SPACES`** ↔ `TAB_SIZE`; **`OUTPUT_DIRECTORY`** ↔ `BASE_SRC_DIR`.
- `X_`-prefixed settings are experimental/hidden — document in an advanced note,
  flagged experimental.

### 1.5 Not-yet-confirmed settings (carry forward)

`TEST_PRODUCTION`/`TEST_EXTENSION` are real (string settings) — used for the
self-test harness; confirm exact behavior from templates/tests. Some settings
may also be consumed in FreeMarker/CTL templates beyond AppSettings; a full
sweep of `src/templates` is a follow-up for the extraction tool.

---

## 2. Grammar keyword / construct set (from CongoCC.ccc)

Confirmed construct keywords (token defs). **Bold = little/no existing blog/wiki
coverage; flag for fresh documentation.**

- Productions/lexical: `TOKEN` (alias `REGULAR_TOKEN`), `SKIP`, `MORE`,
  `UNPARSED` (alias `SPECIAL_TOKEN`), **`CONTEXTUAL`** (contextual tokens),
  `IGNORE_CASE`, `EOF`.
- Lookahead/predicates: `SCAN`, plus up-to-here markers and `=>` (notation).
- Assertions/failure: `ASSERT`, `ENSURE`, `FAIL`.
- Token activation: `ACTIVATE_TOKENS`, `DEACTIVATE_TOKENS`,
  **`ACTIVE_TOKENS`**, **`UNCACHE_TOKENS`**.
- Modularity/codegen: `INCLUDE`, `INJECT`.
- Fault tolerance: `ATTEMPT`, `RECOVER`, **`RECOVER_TO`**, **`ON_ERROR`**.
- Misc: **`FLAG`** (purpose TBD — grep usage in CongoCC.ccc / core when writing).

INCLUDE has built-in **location aliases** (AppSettings:64–83) so `INCLUDE JAVA`
etc. resolve to bundled grammars: `JAVA`, `JAVA_LEXER`, `JAVA_IDENTIFIER_DEF`,
`PYTHON`(+_LEXER/_IDENTIFIER_DEF), `CSHARP`(+_LEXER/_IDENTIFIER_DEF),
`RUST`(+_LEXER/_IDENTIFIER_DEF), `PREPROCESSOR`, `JSON`, `JSONC`, `LUA`.
→ Document these aliases in the INCLUDE/modularity section.

---

## 3. CLI (from live jar, confirms plan)

`java -jar congocc.jar grammarfile` with flags: `-d <dir>`, `-lang
java|python|csharp|rust` (default java), `-n` (suppress newer-version check),
`-p sym[,sym]` (preprocessor symbols), `-q` (quieter). Banner prints build
date. Codegen prints `Outputting: <file>` lines and
`Parser generated successfully.`

---

## 4. Plan §4 corrections

| Plan §4 claim | Verified status |
|---|---|
| `JDK_TARGET` "removed" | **Deprecated no-op** with warning; correct wording |
| `REQUIRE_TOKEN_DECLARATION` new | Confirmed; boolean, default false |
| `DEFAULT_LEXICAL_STATE` reworked | Confirmed as live string setting; semantics changed |
| string-literal auto-labeling | Related to `REQUIRE_TOKEN_DECLARATION`=false (implicit tokens allowed) |

## 5. Follow-ups for later milestones

1. Sweep `src/templates` (CTL/FreeMarker) for any settings consumed outside
   AppSettings before finalizing the settings reference.
2. Determine `FLAG`, `UNCACHE_TOKENS`, `ACTIVE_TOKENS`, `CONTEXTUAL`,
   `RECOVER_TO`, `ON_ERROR` semantics from the core + examples.
3. Confirm exact `JDK_TARGET` warning emission path.
4. The settings-extraction tool (plan §3.2) should parse the three type
   constants in AppSettings.java + the getter defaults to regenerate §1.
