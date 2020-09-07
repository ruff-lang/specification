# Tooling

## Table of Contents

* [`bunny-up`](#bunny-up)
* [The `bunny` command](#the-bunny-command)
* [`bunny format`](#bunny-format)
* [Dependency Manager](#dependency-manager)
* [Carrot Registry](#carrot-registry)
* [Documentation](#documentation)

## `bunny-up`

The `bunny-up` tool is the simplest way to get started with `bunny`, installing the tool is simple and allows installing switching between `bunny` versions. This is based almost entirely off the great [`rustup`](https://rustup.rs/) tool. Install the tool by running the command below on a unix-like system. The source-code for `bunny-up` can be found at [https://github.com/bunny-lang/bunny-up](https://github.com/bunny-lang/bunny-up).

```shell
$ curl --proto '=https' --tlsv1.2 -sSf https://bunny-lang.org/bunny-up.sh | sh
```

Install the language:

```shell
$ bunny-up install  # default to latest 'stable' release
```

Install a specific version:

```shell
$ bunny-up install 0.0.1
```

Use a specific version:

```shell
$ bunny-up use 0.0.1
```

## The `bunny` command

Everything you should need to get up and running with the bunny programming language, from learning to running in production, is included in a single binary: `bunny`. The `bunny` binary includes the core language runtime and standard library, a build tool, a dependency manager, a formatter, a documentation generator, and potentially more.

## `bunny format`

Recursively format all `.bn` files in the current directory tree. Returns the `0` exit code if all files already comply with `bunny` formatting, and returns the `1` exit code when it writes changes.

## Dependency Manager

`bunny` ships with a built in dependency manager to manage `carrot`s. By default, the central carrot registry is used to resolve dependencies. Multiple registries can be used in the `carrots.bn` file to support private `carrot`s. [GitHub Releases](https://docs.github.com/en/github/administering-a-repository/about-releases) can also be used to import `carrot`s that are not in the central registry.

## Carrot Registry

All public `carrot`s are distributed to the centralized `carrot` registry at [https://carrots.bunny-lang.org](https://carrots.bunny-lang.org). Hosting a private `carrot` registry should be easily possible with standard webservers. `carrot`s are uniquely namespaced according to their `git` url, this gives us a uniqueness guarantee for free and discourages bizarre naming schemes to avoid collisions.

A project can declare multiple registry sources in its `carrots.bn` file, and `carrot`s will be searched through the listed registries in order to resolve the package.

## Documentation

Tightly integrated with the Centralized Carrot Registry, [https://docs.bunny-lang.org](https://docs.bunny-lang.org) offers a single source of documentation for the core language, the standard library, and user published `carrot`s.
