Appendix: Legacy Mapping
========================

CongoCC descends from JavaCC (by way of JavaCC 21) but **does not accept legacy
JavaCC syntax** — the old constructs were removed rather than carried forward.
This appendix maps the legacy features to their CongoCC replacements, for
readers coming from JavaCC or JJTree and for converting older grammars.

Tools
-----

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - Legacy tool
     - CongoCC
   * - JJTree
     - No separate preprocessor — tree building is built in and on by default
       (:doc:`../tree-building`).
   * - JJDoc
     - No direct equivalent.

Syntax changes
--------------

.. list-table::
   :header-rows: 1
   :widths: 42 58

   * - Legacy JavaCC
     - CongoCC
   * - ``options { LOOKAHEAD=2; … }``
     - Top-level ``NAME = value;`` settings (:doc:`../grammar-file`).
   * - ``PARSER_BEGIN(X) … PARSER_END(X)``
     - Removed; add code with ``INJECT`` (:doc:`../injection`).
   * - ``void Foo() : {} { Foo() Bar() }``
     - ``Foo : Foo Bar ;`` (:doc:`../productions`).
   * - ``LOOKAHEAD(3)``
     - ``SCAN 3 =>`` or the ``=>||`` marker (:doc:`../disambiguation`).
   * - ``LOOKAHEAD(Foo() Bar())``
     - ``SCAN Foo Bar =>`` or ``Foo Bar =>||``.
   * - ``LOOKAHEAD({ sem })``
     - ``SCAN { sem } =>``.
   * - ``JAVACODE``
     - Removed; use an ordinary production, or ``INJECT`` for hand-written
       methods.
   * - ``TOKEN_MGR_DECLS : { … }``
     - ``INJECT LEXER_CLASS : { … }``.
   * - JJTree ``#Node`` / ``jjtThis`` / ``SimpleNode``
     - The built-in ``#`` node descriptors and the ``Node`` API
       (:doc:`../tree-building`).

Token productions (``<NAME : regex>``) are largely unchanged, though character
classes and complex patterns must be enclosed in ``< … >`` (:doc:`../lexical`).

Settings removed or changed
---------------------------

Many JavaCC options were removed because the behavior is now unconditional, or
replaced by a different mechanism.

.. list-table:: Replaced by a new mechanism
   :header-rows: 1
   :widths: 40 60

   * - Legacy option
     - Replacement
   * - ``COMMON_TOKEN_ACTION``
     - ``TOKEN_HOOK`` method (:doc:`../injection`)
   * - ``NODE_SCOPE_HOOK``
     - ``OPEN_NODE_HOOK`` / ``CLOSE_NODE_HOOK`` methods
   * - ``NODE_EXTENDS``
     - ``INJECT`` an ``extends`` clause
   * - ``TOKEN_FACTORY``
     - ``INJECT`` into the token class
   * - ``TOKEN_MGR_DECLS``
     - ``INJECT`` into the lexer

Now always on (option removed): ``UNICODE_INPUT``, ``KEEP_LINE_COL``,
``ERROR_REPORTING``, ``SANITY_CHECK``.

Removed outright: ``STATIC``, the global ``LOOKAHEAD``,
``CHOICE_AMBIGUITY_CHECK``, ``OTHER_AMBIGUITY_CHECK``, ``FORCE_LA_CHECK``,
``BUILD_PARSER``, ``BUILD_LEXER``, ``DEBUG_PARSER``, ``DEBUG_LEXER``,
``CACHE_TOKENS``, ``USER_DEFINED_TOKEN_MANAGER``, ``USER_CHAR_STREAM``, and
``GRAMMAR_ENCODING`` (input is assumed to be UTF-8). ``JDK_TARGET`` is accepted
but has no effect (:doc:`../settings`).

Converting a legacy grammar
---------------------------

CongoCC does not itself convert legacy grammars. The path is:

1. Run the **syntax converter** included with recent JavaCC 21 builds to
   modernize the old grammar's syntax:

   .. code-block:: console

      $ java -jar javacc-full.jar convert OldGrammar.jj

2. Build the converted grammar with CongoCC, and apply the manual fix-ups that
   conversion does not handle — chiefly import statements and the points below.

Things to expect when moving to CongoCC:

- Tree building always generates **two packages**, one for the parser and one
  for the nodes; code cannot be generated into the default (unnamed) package.
- The ``XXXConstants`` interface no longer exists — token types are the
  ``TokenType`` enum (:doc:`../generated-api`).
- The base node class lives in the node package.
- A grammar that relied on JavaCC's old lookahead quirks may need
  ``LEGACY_GLITCHY_LOOKAHEAD = true;`` to reproduce the original behavior.

A task-oriented walkthrough is in :doc:`/docs/userguide/migration`.
