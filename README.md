# Bunny Language Specification

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
The **Bunny** programming language (called `bunny`, not `bunny lang`) aims to be a simple and general purpose batteries-included language. It is heavily influenced by **Common Lisp**, **Scheme**, **Clojure**, **Ruby**, **Go**, and probably others. The motivation behind the specification is to create a concrete design and incrementally implement the language with different runtimes. The goal is to pick and choose the most developer-friendly features from a bunch of different languages and attempt to bundle them up into a cohesive and enjoyable language.

## Language Features

TODO...

## File Format

Software written in **Bunny** should always be in a [UTF-8](https://en.wikipedia.org/wiki/UTF-8) file with the `.bn` extensions.

## Syntax & Semantics

TODO...

## Libraries (Carrots)

### Overview

Bunny calls libraries carrots, as in "add the postgresql carrot to your dependencies". Creating and distributing carrots should be an integrated part of the `bunny` tooling, much like Ruby with it's incredible [bundler](https://bundler.io/) tool. The tooling should also handle documentation generation, similar to [rustdoc](https://doc.rust-lang.org/rustdoc/what-is-rustdoc.html).

Creating a new `carrot` should be as simple as:

```
$ bunny new carrot <directory_name>
```

This should setup the skeleton for a new carrot. Modify the `carrot_specification.bn` accordingly, and code away.

### Versioning

All `carrot`s should follow [Semantic Versioning](https://semver.org/).

### Distributing

TODO...

## Documentation Generator

TODO...

## Tooling

TODO...

## Conventions

Bunny encourages a set of conventions to make it both more approachable and simplifies tooling by being able to make certain assumptions. The set of conventions for `bunny` are a super-set of a style guide. The ideal would be to have tooling make it hard to break these conventions, and provide enough automation to make the defaults easy to get started with. Strong opinions for conventions is borrowed from Ruby and its wonderful community.

### Avoid Ambiguity

All `bunny` code should avoid ambiguity whenever possible. This applies to file, variable, function, and other names.

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

The directory structure for a carrot (library) is pretty similar, except there's no `src` directory and there's a `carrot_specification.bn` file at the root.

```
.
├── bin
│   └── cli.bn
├── carrot_specification.bn
├── carrots.bn
├── carrots.lock
├── lib
│   └── magic_calculator.bn
└── test
    └── magic_calculator_test.bn
```
