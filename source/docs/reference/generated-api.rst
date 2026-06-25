Generated API
=============

This chapter describes the contract the generated code presents to your
application — the parser, the lexer, tokens, nodes, and the parse exception. The
contract is the same across all four target languages; what differs is naming
and idiom. The names and signatures here follow the Java target; for the
per-language equivalents see the
:doc:`Target Language Guide </docs/targets/targets>`.

Classes and packages
--------------------

For a grammar with base name *Foo*, CongoCC generates a ``FooParser`` and a
``FooLexer`` in the ``PARSER_PACKAGE``, and the node classes in the
``NODE_PACKAGE`` (by default ``<PARSER_PACKAGE>.ast``). All of these names are
configurable; see :doc:`settings`.

The parser
----------

**Construction.** The parser offers constructors for the common input sources —
an in-memory ``CharSequence`` or a file ``Path`` — each optionally taking a
name for the input source (used in error messages), and one taking a
pre-built lexer:

.. code-block:: java

   new FooParser(CharSequence content)
   new FooParser(String inputSource, CharSequence content)
   new FooParser(Path path)                       // throws IOException
   new FooParser(String inputSource, Path path)
   new FooParser(FooLexer lexer)

**Parsing.** Each production becomes a method of the same name. Call the one
that is your start symbol to parse; a production declared with a return type
returns its value:

.. code-block:: java

   parser.Document();          // parse, building the tree
   int n = parser.Sum();       // a production with a return type

**The result.** After parsing, ``rootNode()`` returns the root :ref:`Node
<docs/reference/generated-api:Nodes>` of the syntax tree.

**Run-time controls.** A handful of methods adjust behavior at run time:

- ``setBuildTree(boolean)`` / ``getBuildTree()`` and ``isTreeBuildingEnabled()``
  — turn tree building on or off for a parse.
- ``setTokensAreNodes(boolean)`` and ``setUnparsedTokensAreNodes(boolean)`` —
  the run-time counterparts of the tree settings.
- ``setParserTolerant(boolean)`` / ``isParserTolerant()`` — switch
  :doc:`fault-tolerant <fault-tolerance>` parsing on or off.
- ``cancel()`` / ``isCancelled()`` — cooperatively cancel a long-running parse.
- ``getNextToken()`` and ``getToken(int index)`` — direct access to the token
  stream, mainly for use inside grammar actions.

The lexer
---------

``FooLexer`` produces the token stream from the input. It is usually driven by
the parser and not used directly, but it can be constructed on its own and
passed to the parser's lexer constructor when you need to tokenize without
parsing.

Tokens
------

Every token is an instance of the token class (``Token`` by default, settable
with ``BASE_TOKEN_CLASS``). A token is also a :ref:`Node
<docs/reference/generated-api:Nodes>`, so it carries position information and
fits in the tree. Its members include:

- ``getType()`` — the token's ``TokenType``.
- ``getSource()`` / ``getImage()`` — the matched text.
- ``getBeginLine()``, ``getBeginColumn()``, ``getEndLine()``, ``getEndColumn()``
  and the offset forms ``getBeginOffset()`` / ``getEndOffset()`` — the
  token's position in the input.
- ``getNext()`` / ``getPrevious()`` — the adjacent tokens in the stream.
- ``isUnparsed()`` — whether this is an unparsed (special) token such as a
  comment.

``TokenType`` is a generated enumeration with one value per declared token
type, plus ``EOF``. Using an enum rather than integer constants means token
comparisons are type-checked at compile time.

Nodes
-----

Every syntax-tree node implements the ``Node`` interface. Its traversal and
position members are the working surface for consuming a tree; the most used
are summarized in :doc:`tree-building` (``children()``, ``descendants()``,
``firstChildOfType(...)``, ``getType()``, ``getParent()``, ``getSource()``,
``dump()``). Three nested types complete the model:

- ``Node.NodeType`` — the common supertype of ``TokenType`` and the node-type
  enumeration, so a node's ``getType()`` covers both tokens and productions.
- ``Node.Visitor`` — the reflective visitor base class (see
  :doc:`tree-building`).
- ``Node.CodeLang`` — the enumeration of target languages (``JAVA``,
  ``PYTHON``, ``CSHARP``, ``RUST``).

The parse exception
-------------------

When the input does not match the grammar, the parser throws a
``ParseException``. By default it is an **unchecked** exception (it extends the
language's runtime-exception type), so callers are not forced to declare or
catch it; set ``USE_CHECKED_EXCEPTION`` to make it checked instead. It carries:

- ``getMessage()`` — a human-readable description, including the position and
  what was expected; the stack trace includes locations in the *grammar*, not
  just the generated code.
- ``getLocation()`` / ``getToken()`` — the node/token where parsing failed.
- ``hitEOF()`` — whether the failure was an unexpected end of input.

When :doc:`fault-tolerant <fault-tolerance>` parsing is enabled, errors are
recovered from and recorded rather than thrown, and the returned tree may
contain nodes flagged as incomplete.
