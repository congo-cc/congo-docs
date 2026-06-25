Invocation
==========

CongoCC is run from its executable JAR. This chapter is the reference for CongoCC's
command line and runtime environment. For JAR installation instructions, see
:doc:`/docs/userguide/installation`.

Basic usage
-----------

.. code-block:: console

   $ java -jar congocc.jar [flags] grammarfile

The single non-flag argument is the grammar file to process. With no arguments
at all, CongoCC prints its banner and the usage message and exits.

Command-line flags
------------------

``-d <directory>``
   Output directory for the generated files, absolute or relative to the
   grammar file's location. If omitted, files are written relative to the
   grammar file. The directory is created if it does not exist.

``-lang <language>``
   Target language: ``java`` (the default), ``python``, ``csharp``, or
   ``rust``.

``-p <symbols>``
   One or more comma-separated preprocessor symbols (no spaces), each either a
   bare ``NAME`` or ``NAME=value``. A bare name is given the value ``1``. These
   symbols drive the preprocessor (see :doc:`grammar-file`).

``-n``
   Suppress the check for a newer release (below).

``-q``
   Request quieter output.

Flags may be written with one or two leading dashes. An unrecognized flag is
reported and ignored rather than treated as an error. The old ``-jdk`` flag has
been removed; if present it is ignored with a notice.

The newer-version check
-----------------------

By default, each run checks online whether a newer ``congocc.jar`` is available
and, if so, offers to download it. Pass ``-n`` to skip this — appropriate when
working offline or in automated builds.

Overriding settings from the environment
----------------------------------------

Settings in the grammar file take the form ``SETTING=value``. 
See the :doc:`settings` for the list of all available settings. 
Grammar file settings can be overridden by environment variables or by command-line flags. 
An environment variable named ``CONGOCC_<SETTING>`` overrides that setting in the grammar file. 
A command-line flag of the form ``-p SETTING=value`` does the same from the command line. 
When a setting is specified in more than one place, the order of priority is:

   grammar file < environment variable < command line

This is convenient for build scripts that need to redirect output or flip a
flag without editing the grammar.

Exit status
-----------

CongoCC exits ``0`` on success and with a non-zero status on failure.  Failures include usage
errors (no input file, an unreadable grammar, a bad flag argument) or errors
detected in the grammar itself. On success it lists each generated file and
finishes with ``Parser generated successfully.``

Converting legacy JavaCC grammars
---------------------------------

CongoCC does not itself convert legacy JavaCC or JavaCC 21 grammars; it only
reads current CongoCC syntax. The migration path, including the separate
converter tool used to modernize older grammars, is described in
:doc:`appendices/legacy` and :doc:`/docs/userguide/migration`.

Build integration
-----------------

For driving CongoCC from Ant, Maven, Gradle, or CI — and for regenerating
deterministically — see :doc:`/docs/userguide/howto/build-integration`, and the
:doc:`Target Language Guide </docs/targets/targets>` for per-language build
details.
