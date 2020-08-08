# Ruff Language Specification

The Ruff programming language (called `ruff`, not `ruff-lang`) aims to be a simple, convenient, general purpose, batteries-included language. It is heavily influenced by Common Lisp, Scheme, Clojure, Ruby, Go, and probably others. The motivation behind the specification is to create a concrete design and incrementally implement the language with different runtimes. The goal is to pick and choose the most developer-friendly features from a bunch of different languages and attempt to bundle them up into a cohesive and enjoyable language.

## File Format

Software written in Ruff should always be in a [UTF-8](https://en.wikipedia.org/wiki/UTF-8) file with the `.rf` extension.
