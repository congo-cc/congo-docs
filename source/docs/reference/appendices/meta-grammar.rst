Appendix: Grammar of the Grammar
================================

This appendix gives the syntax of the CongoCC grammar language itself, in EBNF.
It is a curated, readable presentation meant for quick reference; the
authoritative and complete definition is the self-hosted grammar that CongoCC
uses to parse ``.ccc`` files (``CongoCC.ccc`` in the source distribution), which
necessarily covers details elided here.

Notation
--------

.. code-block:: text

   =        defines a rule
   |        alternatives
   { x }    zero or more repetitions of x
   [ x ]    x is optional
   ( x )    grouping
   "x"      the literal text x
   Name     an identifier;  String  a quoted string;  Int  an integer literal

Grammar file
------------

.. code-block:: text

   GrammarFile    = { Setting } { GrammarElement }
   GrammarElement = TokenProduction | Production | Injection | Inclusion
   Setting        = Name [ "=" Value ] ";"
   Value          = "true" | "false" | Int | String | Name { "," Name }
   Inclusion      = "INCLUDE" Location { "!" Location } [ ";" ]
   Location       = String | Name                 // a path, or a built-in alias

A setting written as a bare name is a boolean set to true. Comments use ``//``
and ``/* … */``. The preprocessor directives (``#if`` and friends) are handled
before this grammar applies; see :doc:`../grammar-file`.

Token productions
-----------------

.. code-block:: text

   TokenProduction = [ States ] TokenKind [ "[" "IGNORE_CASE" "]" ] [ "#" Name ] ":"
                     RegexpSpec { "|" RegexpSpec } ";"
   States          = "<" ( "*" | Name { "," Name } ) ">"
   TokenKind       = "TOKEN" | "UNPARSED" | "SKIP" | "MORE" | "CONTEXTUAL"

   RegexpSpec      = String
                   | "<" [ ( "#" | "?" ) Name ":" ] Regexp ">"
                     [ "#" Name ] [ Action ] [ ":" Name ]
   Regexp          = RegexpSeq { "|" RegexpSeq }
   RegexpSeq       = RegexpUnit { RegexpUnit }
   RegexpUnit      = String
                   | "<" Name ">"                  // reference to another token/regexp
                   | CharClass
                   | "(" Regexp ")" [ "*" | "+" | "?" | "{" Int [ "," [ Int ] ] "}" ]
   CharClass       = [ "~" ] "[" [ CharRange { "," CharRange } ] "]"
   CharRange       = Char [ "-" Char ]

In a ``RegexpSpec``, a leading ``#`` marks a private regular expression and a
leading ``?`` marks a lazy token; a trailing ``: Name`` switches lexical state
after the match.

Productions
-----------

.. code-block:: text

   Production     = [ Access ] [ ReturnType ] Name [ "(" Params ")" ] [ "throws" Names ]
                    [ NodeDescriptor ] [ "RECOVER_TO" Expansion ] ":"
                    [ Name ":" ] Expansion ";"
   Access         = "public" | "private" | "protected"
   NodeDescriptor = "#" ( Name | "void" | "abstract" | "interface" )
                    [ "(" [ CmpOp ] Expression ")" ]
   CmpOp          = ">" | ">=" | "<" | "<="

The optional ``Name ":"`` after the first colon sets the production's starting
lexical state.

Expansions
----------

.. code-block:: text

   Expansion      = Sequence { "|" Sequence }
   Sequence       = [ Lookahead ] { ExpansionUnit }
   ExpansionUnit  = Terminal
                  | NonTerminal
                  | "(" Expansion ")" [ "*" | "+" | "?" ]
                  | "[" Expansion "]"
                  | Action
                  | Assertion
                  | "FAIL" [ Message ]
                  | "ATTEMPT" Expansion "RECOVER" ( "(" Expansion ")" | Action )
                  | "UNCACHE_TOKENS"
                  | TokenActivation "(" Expansion ")"
   Terminal       = [ Assignment ] ( String | "<" Name ">" | "<" "EOF" ">" )
                    [ "!" ] [ UpToHere ]
   NonTerminal    = [ Assignment ] Name [ "(" Args ")" ] [ "!" ] [ UpToHere ]
   TokenActivation= ( "ACTIVATE_TOKENS" | "DEACTIVATE_TOKENS" | "ACTIVE_TOKENS" )
                    Name { [ "," ] Name }
   UpToHere       = "=>|" ( "|" | "+" Digit )
   Action         = "{" TargetLanguageCode "}"

Lookahead and assertions
------------------------

.. code-block:: text

   Lookahead      = ( "SCAN" | Int ) [ Int ]
                    { "{" Expression "}" }
                    [ LookBehind ]
                    [ [ "~" ] Expansion ] "=>"
   LookBehind     = [ "~" ] ( "\" | "/" ) PathElem { ( "\" | "/" ) PathElem }
   PathElem       = [ "~" ] Name | "." | "..."
   Assertion      = ( "ASSERT" | "ENSURE" )
                    ( "{" Expression [ ":" Message ] "}" | [ "~" ] "(" Expansion ")" )

Injection and inclusion
-----------------------

.. code-block:: text

   Injection  = "INJECT" Name ":" [ Imports ] [ "extends" Type ]
                  [ "implements" Types ] [ "{" Members "}" ]
              | "INJECT" ":" "{" CompilationUnit "}"     // top-level code
              | "INJECT" [ Name ] "{" Code "}"           // raw block

The injection target ``Name`` may be a node type, a magic class name
(``PARSER_CLASS``, ``LEXER_CLASS``, ``BASE_NODE_CLASS``, ``BASE_TOKEN_CLASS``),
or the ``Node`` interface; see :doc:`../injection`.
