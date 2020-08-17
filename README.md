# Bunny Language Specification

**NOTE: This is a work in-progress.**

```
                                                                 
                     @@@@@             @@@@@@                    
                   @@     @@@        @@.     @@                  
                   @@     @@@        @@.     @@                  
                   @@     @@@        @@.     @@                  
                   @@     @@@        @@.     @@                  
                   @@     @@@        @@.     @@                  
                   @@     @@@        @@.     @@                  
                   @@     @@@        @@.     @@                  
                   @@        @@@@@@@@        @@                  
                @@@                            @@@               
                @@@                            @@@               
             @@@        @@.            @@@        @@             
             @@@        @@.            @@@        @@             
             @@@                @@                @@             
             @@@             @@@  @@@             @@             
                @@@                            @@@               
                @@@                            @@@               
                   @@@@@                  @@@@@                  
                        @@@@@@@@@@@@@@@@@@                       
                                                                 
```

## Table of Contents
- [Overview](#overview)
- [Language Features](#language-features)
- [Syntax & Semantics](#syntax--semantics)
  * [Atoms](#atoms)
  * [Cons Cells](#cons-cells)
  * [Lists](#lists)
- [Libraries (Carrots)](#libraries-carrots)
  * [Overview](#overview-1)
  * [Creating a Library](#creating-a-library)
  * [Adding a Dependency](#adding-a-dependency)
  * [Versioning](#versioning)
  * [Distributing](#distributing)
- [Documentation Generator](#documentation-generator)
- [Tooling](#tooling)
  * [`bunny-up`](#bunny-up)
  * [The `bunny` command](#the-bunny-command)
  * [`bunny format`](#bunny-format)
  * [Dependency Manager](#dependency-manager)
  * [Carrot Registry](#carrot-registry)
  * [Documentation](#documentation)
- [Conventions & Style Guide](#conventions--style-guide)
  * [File Format](#file-format)
  * [Directory Structure](#directory-structure)
  * [Code Format](#code-format)
  * [Avoid Ambiguity](#avoid-ambiguity)
  * [Document Carrot Interfaces](#document-carrot-interfaces)

## Overview

**Bunny** (_bunny_, not _bunny lang_) aims to be a simple, practical, and fun general purpose dynamic programming language. 

It is heavily influenced by **Common Lisp**, **Scheme**, **Clojure**, **Ruby**, **Go**, and probably others. The motivation behind the specification is to create a concrete design and incrementally implement the language with different runtimes. The goal is to pick and choose the most developer-friendly features from a bunch of different languages and attempt to bundle them up into a cohesive and enjoyable language.

## Language Features

- **Minimal and Modern Syntax**, inspired by the elegance of Scheme and readability of Ruby.
- **Functional**, as in functions are values.
- **Tail-Call Optimized**, recursion without the overhead.
- **Garbage-Collected**, using a mark-and-sweep algorithm.
- **Macros**.
- **Continuations**, a simple interface to `call/cc`.
- **Concurrent**, in the spirit of Go with coroutines and channels (built on top of continuations).
- **Batteries Included**, to be a practical and productive tool (see also: https://github.com/bunny-lang/stdlib).
- **Integrated Tooling**, `bunny` ships as a single binary with all the tools needed to be productive.

## Syntax & Semantics

### Atoms

An `atom` is any singlular piece of data that is not a pair. Bunny has a couple of different kinds of atoms. 

|**Atom** |**Semantic Meaning**|
|---------|--------------------|
|symbol   | A symbol is any non-reserved word, for example `foo` is a symbol and so is `+` |
|boolean  | `true` is a symbol representing the truthy boolean. `false` is an aliased symbol representing falsity (the underlying implementation for falsity is `null`) |
|integer  | Any number, for example `1` or `19` |
|float    | A floating point integer, for example `3.14159265` |
|character| Any letter prepended with a backslash, for example `\a` represents the first letter of the english alphabet. |

### Cons Cells

The fundamental data type in Bunny is a pair of two things, also known as a cons cell. A first element (called `head`) and a second element (called `tail`) are all that make up a cons cell.

The syntax for a cons cell is `(cons <head> <tail>)`, for example the pair for two symbols, `a` and `b` is written as `(cons a b)`. Many lisps use the dotted-pair notation to represent cons cells, for example `(<head> . <tail>)` is equivalent to `(cons <head> <tail>)`. Bunny differs here, instead reserving the `.` symbol for other operations (note: this is not finalized but may be a reversed decision).

### Lists

A list is an arbitrary length sequence of cons cells. The `tail` of each cons cell holds the next cons cell, or contains `null` as a special symbol designating sequence termination. Any data structure formed with cons cells is a list.

The underlying representation for a list is a cons pair as described below:

```
(1 2 3) => (cons 1 (cons 2 (cons 3 null)))
```

| **Examples** | **Semantic Meaning** |
|--------------|--------------------|
|`()`          | an empty list |
|`null`        | an empty list, equivalent to `()` |
|`(a)`         | a list containing the symbol `a` |
|`(a ())`      | equivalent to `(a)`, the symbol `a` and the empty list `()` |
|`(a null)`    | equivalent to `(a)`, the symbol `a` and empty list `null` |
|`(a (b c))`   | two element list of the symbol `a` and the list `(b c)` |

`head` and `tail` are also functions that can be applied to lists, for example, `(head '(1 2 3))` evaluates to `1` and `(tail '(1 2 3))` evaluates to `(2 3)` (note: the `'` is explained further later). It's possible to get the `nth` item using the function `nth`, e.g. `(nth 2 '(1 2 3)` evaluates to `3`.

Given an empty list `()`, `(head '())` is `()` and `(tail '())` is `()`. In other words, the `head` and `tail` of an empty list is `null`.

## Libraries (Carrots)

### Overview

Bunny calls libraries carrots, as in "add the postgresql carrot to your dependencies". Creating and distributing carrots should be an integrated part of the `bunny` tooling, much like Ruby with it's incredible [bundler](https://bundler.io/) tool. The tooling should also handle documentation generation, similar to [rustdoc](https://doc.rust-lang.org/rustdoc/what-is-rustdoc.html).

### Creating a Library

Creating a new `carrot` should be as simple as:

```
$ bunny new carrot <carrot_name:required>
```

This should setup a directory `<carrot_name>` with the skeleton for a new carrot. Modify the `carrot_spec.bn` accordingly, and code away.

### Adding a Dependency

All project or library dependencies should be added to the `carrots.bn` file either manually or by invoking the `bunny` command.

```
$ bunny add carrot <carrot_name:required> <carrot_version:optional>
```

This will automatically add the dependency to the `carrots.bn` file.

### Versioning

All `carrot`s should follow [Semantic Versioning](https://semver.org/). For example, the first stable release of Bunny will be `v1.0.0`

### Distributing

`carrot`s are encouraged to be published and distributed through the [carrot registry](#carrot-registry), which is the default built into the dependency manager. It is also possible to distribute `carrot`s through [GitHub Releases](https://docs.github.com/en/github/administering-a-repository/about-releases).

## Documentation Generator

TODO...

## Tooling

### `bunny-up`

The `bunny-up` tool is the simplest way to get started with `bunny`, installing the tool is simple and allows installing switching between `bunny` versions. This is based almost entirely off the great [`rustup`](https://rustup.rs/) tool. Install the tool by running the command below on a unix-like system.

```
$ curl --proto '=https' --tlsv1.2 -sSf https://bunny-lang.org/bunny-up.sh | sh
```

Install the language:

```
$ bunny-up install  # default to latest 'stable' release
```

Install a specific version:

```
$ bunny-up install 0.0.1
```

Use a specific version:

```
$ bunny-up use 0.0.1
```

### The `bunny` command

Everything you should need to get up and running with the bunny programming language, from learning to running in production, is included in a single binary: `bunny`. The `bunny` binary includes the core language runtime and standard library, a build tool, a dependency manager, a formatter, a documentation generator, and potentially more.

### `bunny format`

Recursively format all `.bn` files in the current directory tree. Returns the `0` exit code if all files already comply with `bunny` formatting, and returns the `1` exit code when it writes changes.

### Dependency Manager

`bunny` ships with a built in dependency manager to manage `carrot`s. By default, the central carrot registry is used to resolve dependencies. Multiple registries can be used in the `carrots.bn` file to support private `carrot`s. [GitHub Releases](https://docs.github.com/en/github/administering-a-repository/about-releases) can also be used to import `carrot`s that are not in the central registry.

### Carrot Registry

All public `carrot`s are distributed to the centralized `carrot` registry at https://carrots.bunny-lang.org. Hosting a private `carrot` registry should be easily possible with standard webservers. `carrot`s are uniquely namespaced according to their `git` url, this gives us a uniqueness guarantee for free and discourages bizarre naming schemes to avoid collisions.

A project can declare multiple registry sources in its `carrots.bn` file, and `carrot`s will be searched through the listed registries in order to resolve the package.

### Documentation

Tightly integrated with the Centralized Carrot Registry, https://docs.bunny-lang.org offers a single source of documentation for the core language, the standard library, and user published `carrot`s.

## Conventions & Style Guide

Bunny encourages a set of conventions to make it both more approachable and simplifies tooling by being able to make certain assumptions. The ideal would be to have tooling make it hard to break these conventions, and provide enough automation to make the defaults easy to get started with. Strong opinions for conventions is borrowed from Ruby and its wonderful community.

### File Format

Software written in **Bunny** should always be in a [UTF-8](https://en.wikipedia.org/wiki/UTF-8) file with the `.bn` extensions. File names should always be lower-case and `snake_case`, e.g. `file_format.bn`.

### Directory Structure

Here's an example directory structure for a hypothetical project.

```
.
├── bin
│   ├── cli.bn
│   └── server.bn
├── carrots.bn
├── carrots.lock
├── lib
│   └── utils.bn
├── src
│   └── application.bn
└── test
    └── application_test.bn
```

- `bin` (optional) should contain entrypoints to code, for example a `cli` launcher or `server` launcher.
- `carrots.bn` should describe the dependencies for the project.
- `carrots.lock` is the generated lock file with the dependency graph based on `carrots.bn`.
- `lib` (optional) should contain any utilities or shared code for the project.
- `src` contains all the application specific code for the project.
- `test` contains tests for the project.

The directory structure for a carrot (library) is pretty similar, except there's no `src` directory and there's a `carrot_spec.bn` file at the root.

```
.
├── bin
│   └── cli.bn
├── carrot_spec.bn
├── carrots.bn
├── carrots.lock
├── lib
│   └── magic_calculator.bn
└── test
    └── magic_calculator_test.bn
```

### Code Format

Code structure and formatting should not be an argument. Bunny takes Go's approach by providing a standardized code formatter with the core tooling.

```
$ bunny format  # or aliased 'bunny fmt'
```

### Avoid Ambiguity

All `bunny` code should avoid ambiguity whenever possible. This applies to file, variable, function, and other names.

### Document Carrot Interfaces

Each `carrot` should document its public interface using the documentation comment syntax above function definitions. When distributing `carrot`s, the documentation generator can publish the documentation in the centralized `carrot` repository and provide a uniform way to browse documentation.
