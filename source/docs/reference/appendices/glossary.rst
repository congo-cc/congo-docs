Appendix: Glossary
==================

.. glossary::

   abstract syntax tree
   AST
      The tree of :term:`nodes <node>` a parser produces, representing the
      structure of the parsed input. See :doc:`../tree-building`.

   choice point
      A place in a grammar where the parser must decide between alternatives —
      a ``|`` alternation, an optional ``[ … ]``, or a loop ``( … )*`` /
      ``( … )+``. See :doc:`../disambiguation`.

   contextual predicate
      A look-behind condition that selects an alternative based on which
      productions are currently on the parse stack, rather than on the upcoming
      tokens. See :doc:`../disambiguation`.

   contextual token
      A token, declared with the ``CONTEXTUAL`` kind, that the lexer produces
      only where the parser expects it. See :doc:`../tokenization-advanced`.

   expansion
      The right-hand side of a :term:`production` — the pattern of tokens,
      non-terminals, and operators it matches. See :doc:`../productions`.

   fault-tolerant parsing
      A mode in which the parser recovers from input that does not match the
      grammar and still produces a tree, rather than stopping at the first
      error. See :doc:`../fault-tolerance`.

   FIRST set
      The set of tokens that can legally begin a given expansion; the parser
      uses it to choose alternatives by the next token. See
      :doc:`../disambiguation`.

   grammar
      The complete description of a language, written in a ``.ccc`` file, from
      which CongoCC generates a parser.

   hook
      A specially named method (such as ``TOKEN_HOOK``) that, when injected into
      the parser, CongoCC wires into the generated code. See
      :doc:`../injection`.

   injection
      Adding your own code — methods, fields, supertypes — to the classes
      CongoCC generates, by means of the ``INJECT`` statement. See
      :doc:`../injection`.

   lazy token
      A token, marked with ``?``, that matches the shortest text satisfying its
      pattern instead of the longest. See :doc:`../tokenization-advanced`.

   lexer
      The component that turns input characters into a stream of
      :term:`tokens <token>`; also called the tokenizer or scanner. See
      :doc:`../lexical`.

   lexical state
      A mode the lexer is in, determining which token types it can currently
      match. The starting state is ``DEFAULT``. See :doc:`../lexical`.

   lookahead
      Information about the upcoming input that the parser uses to choose
      between alternatives at a :term:`choice point`. See
      :doc:`../disambiguation`.

   node
      An element of the syntax tree. Both productions and tokens can produce
      nodes. See :doc:`../tree-building`.

   non-terminal
      A reference to a :term:`production` within an expansion.

   parser
      The component that consumes the token stream according to the grammar's
      productions and builds the syntax tree.

   private regular expression
      A named pattern, declared with ``<#NAME : … >``, that can be referenced
      from other patterns but is not itself a token type. See
      :doc:`../lexical`.

   production
      A named grammar rule. CongoCC generates one parser method per production.
      See :doc:`../productions`.

   recursive descent
      The parsing technique CongoCC uses, in which each production is realized
      as a function that calls the functions for the productions it references.

   smart node creation
      The default behavior whereby a production builds a node only when it would
      have more than one child, suppressing trivial one-child wrappers. See
      :doc:`../tree-building`.

   target language
      The programming language CongoCC generates code in: Java, Python, C#, or
      Rust. See the :doc:`Target Language Guide </docs/targets/targets>`.

   terminal
      A :term:`token` as it appears in an expansion — a string literal or a
      ``<NAME>`` reference.

   token
      An indivisible lexical unit produced by the lexer, such as a number,
      identifier, or punctuation mark. See :doc:`../lexical`.

   token type
      The category of a token, declared in a token production and represented at
      run time by a value of the generated ``TokenType`` enumeration. See
      :doc:`../generated-api`.

   unparsed token
      A token, declared with ``UNPARSED`` (or ``SPECIAL_TOKEN``), that is kept
      but not passed to the parser — typically a comment. See
      :doc:`../lexical`.

   up-to-here marker
      The ``=>||`` notation that tells the parser to scan the input up to that
      point when deciding whether to take an alternative. See
      :doc:`../disambiguation`.
