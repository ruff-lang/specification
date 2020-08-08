# Ruff Language Specification

The Ruff programming language (called `ruff`, not `ruff-lang`) aims to be a simple, convenient, general purpose, batteries-included dynamic language. It is heavily influences by the likes of Common Lisp, Scheme, Clojure, and Ruby. The motivation behind the specification is to create a concrete design and incrementally implement the language with different runtimes.

## File Format

Software written in Ruff should always be in a [UTF-8](https://en.wikipedia.org/wiki/UTF-8) file with the `.rf` extension.
