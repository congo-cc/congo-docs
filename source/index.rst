.. CongoCC documentation master file.

CongoCC Documentation
=====================

CongoCC is a recursive-descent parser generator that produces parsers in
**Java, Python, C#, and Rust** from a single grammar. This documentation is
organized into three guides:

* the **Reference Manual** — the complete, precise description of the grammar
  language, command-line interface, settings, and generated API;
* the **User Guide** — tutorials, how-to guides, and explanations for getting
  real parsing work done;
* the **Target Language Guide** — everything specific to generating and using
  parsers in each supported target language.

.. note::

   CongoCC ships as a continuously updated tool rather than as numbered
   releases. Where a feature's availability matters, the page marks it with a
   date — for example, *Added 2026-06-19*.

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
