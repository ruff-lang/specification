# Bunny Language Specification

![Bunny][logo-image] **Bunny** (_bunny_, not _bunny lang_) aims to be a simple, practical, and fun general purpose dynamic programming language. It is inspired by a collection of languages including classic Lisp dialects like **Common Lisp** and **Scheme**, modern Lisp-inspired languages like **Clojure**, dynamic and general purpose languages like **Ruby** and **Python**, and elegant concurrent languages like **Go**.

The motivation behind the specification is to produce a concrete language design with wish-list features from it's inspirations. The goal is to bundle the most developer-friendly features into a cohesive and enjoyable language.

## Language Features

- **Minimal and Modern Syntax**, aiming to be elegant, expressive, and readable.
- **Functional**, as in functions are values.
- **Tail-Call Optimized**, recursion without the overhead.
- **Garbage-Collected**, using a mark-and-sweep algorithm.
- **Hygienic Macros**.
- **Continuations**, as a first-class language feature (`call/cc`).
- **Concurrent**, in the spirit of Go with coroutines and channels (built on top of continuations).
- **Batteries Included**, to be a practical and productive tool (see also: https://github.com/bunny-lang/stdlib).
- **Integrated Tooling**, `bunny` ships as a single binary with all the tools needed to be productive.

## Documents

- [Syntax and Semantics](./syntax-and-semantics.md)
- [Libraries (Carrots)](./libraries.md)
- [Documentation Generator](./documentation-generator.md)
- [Tooling](./tooling.md)
- [Conventions and Style Guide](./conventions-and-style-guide.md)

[logo-image]: https://camo.githubusercontent.com/f37c0d5d51478a1c4e9985f5773902a9fe2c18de/68747470733a2f2f692e70696e696d672e636f6d2f343734782f66302f31372f37362f66303137373666333334376164646535363465303266373136636234376262392d2d6d7974686963616c2d6372656174757265732d706978656c2d6172742e6a7067
