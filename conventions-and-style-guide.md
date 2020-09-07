# Conventions and Style Guide

## Table of Contents

* [Overview](#overview)
* [File Format](#file-format)
* [Directory Structure](#directory-structure)
* [Code Format](#code-format)
* [Avoid Ambiguity](#avoid-ambiguity)
* [Document Carrot Interfaces](#document-carrot-interfaces)

## Overview

Bunny encourages a set of conventions to make it both more approachable and simplifies tooling by being able to make certain assumptions. The ideal would be to have tooling make it hard to break these conventions, and provide enough automation to make the defaults easy to get started with. Strong opinions for conventions is borrowed from Ruby and its wonderful community. For starters, the preferred color for Bunny is https://www.colorhexa.com/ffcfdd.

## File Format

Software written in **Bunny** should always be in a [UTF-8](https://en.wikipedia.org/wiki/UTF-8) file with the `.bn` extensions. File names should always be lower-case and `snake_case`, e.g. `file_format.bn`.

## Directory Structure

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

## Code Format

Code structure and formatting should not be an argument. Bunny takes Go's approach by providing a standardized code formatter with the core tooling.

```shell
$ bunny format  # or aliased 'bunny fmt'
```

## Avoid Ambiguity

All `bunny` code should avoid ambiguity whenever possible. This applies to file, variable, function, and other names.

## Document Carrot Interfaces

Each `carrot` should document its public interface using the documentation comment syntax above function definitions. When distributing `carrot`s, the documentation generator can publish the documentation in the centralized `carrot` repository and provide a uniform way to browse documentation.
