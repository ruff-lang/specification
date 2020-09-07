# Libraries (Carrots)

## Table of Contents

* [Overview](#overview)
* [Creating a Library](#creating-a-library)
* [Adding a Dependency](#adding-a-dependency)
* [Using a Library](#using-a-library)
* [Versioning](#versioning)
* [Distributing](#distributing)

## Overview

Bunny calls libraries carrots, as in "add the postgresql carrot to your dependencies". Creating and distributing carrots should be an integrated part of the `bunny` tooling, much like Ruby with it's incredible [bundler](https://bundler.io/) tool. The tooling should also handle documentation generation, similar to [rustdoc](https://doc.rust-lang.org/rustdoc/what-is-rustdoc.html).

## Creating a Library

Creating a new `carrot` should be as simple as:

```shell
$ bunny carrot new <carrot_name:required> --bin=cli  # example to add an optional binary executable entrypoint to the carrot
```

This should setup a directory `<carrot_name>` with the skeleton for a new carrot. Modify the `carrot_spec.bn` accordingly, and code away. Other projects can use this code now by adding `<carrot_name>` to its `carrots.bn` file and importing `<carrot_name>` in code. Below is what the directory structure will look like.

```shell
.
├── bin          # Optional: The binary directory can hold any code specific to binary executable entry points
│   └── cli.bn   #           that might be shipped with a carrot. This example shows a cli entrypoint.
├── carrot_spec.bn
├── carrots.bn
├── carrots.lock
├── lib
│   └── <carrot_name>.bn
└── test
    └── <carrot_name>_test.bn
```

`<carrot_name>.bn` file:

```clojure
(ns <carrot_name>)

(def magic-number 42)

(defn foo (argument)
  "Returns whatever argument is passed to the function."
  argument)
```

## Adding a Dependency

All project or library dependencies should be added to the `carrots.bn` file either manually or by invoking the `bunny` command.

```shell
$ bunny carrot add <carrot_name:required> <carrot_version:optional>
```

This will automatically add the dependency to the `carrots.bn` file.

## Using a Library

Functions in `<carrot_name>.bn` can be used after importing, below is usage based on the above example for [Creating a Library](#creating-a-library).

```clojure
(use (<carrot_name>))

(<carrot_name>/foo "rabbits are cool and stuff")
(<carrot_name>/magic-number)  ; This will evaluate to 42.

;; Optionally, one could import as an alias.
(use (<carrot_name> :as cn))

(cn/foo "this is much shorter!")
(cn/magic-number)        ; Still evaluates to 42.
(mut cn/magic-number 1)  ; Mutate the variable globally.
(cn/magic-number)        ; Now evaluates to 1.
```

## Versioning

All `carrot`s should follow [Semantic Versioning](https://semver.org/). For example, the first stable release of Bunny will be `v1.0.0`.

## Distributing

`carrot`s are encouraged to be published and distributed through the [carrot registry](#carrot-registry), which is the default built into the dependency manager. It is also possible to distribute `carrot`s through [GitHub Releases](https://docs.github.com/en/github/administering-a-repository/about-releases).
