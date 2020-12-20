## Table Of Contents

* [Syntax and Semantics](#syntax-and-semantics)
  * [Atoms](#atoms)
  * [Cons Cells](#cons-cells)
  * [Lists](#lists)
  * [Variables](#variables)
  * [Functions](#functions)
  * [Conditionals](#conditionals)
  * [Namespaces](#namespaces)
  * [Comments](#comments)
* [Libraries (Carrots)](#libraries-carrots)
  * [Creating a Library](#creating-a-library)
  * [Adding a Dependency](#adding-a-dependency)
  * [Using a Library](#using-a-library)
  * [Versioning](#versioning)
  * [Distributing](#distributing)
* [Tooling](#tooling)
  * [`bunny-up`](#bunny-up)
  * [The `bunny` command](#the-bunny-command)
  * [`bunny format`](#bunny-format)
  * [Dependency Manager](#dependency-manager)
  * [Carrot Registry](#carrot-registry)
  * [Documentation Generation](#documentation-generator)
  * [Centralized Documentation](#centralized-documentation)
* [Conventions and Style Guide](#conventions-and-style-guide)
  * [File Format](#file-format)
  * [Directory Structure](#directory-structure)
  * [Code Format](#code-format)
  * [Avoid Ambiguity](#avoid-ambiguity)
  * [Document Carrot Interfaces](#document-carrot-interfaces)

---

## Syntax and Semantics

### Atoms

An `atom` is any singlular piece of data that is not a pair. Bunny has a couple of different kinds of atoms.

|**Atom** |**Semantic Meaning**|
|---------|--------------------|
|symbol   | A symbol is a word or an operator, for example `foo` is a symbol and so is `+` |
|keyword  | A keyword is like a symbol that is exported and evaluates to itself, for example `:foo` is a keyword |
|boolean  | `true` is a symbol representing the truthy boolean. `false` is an aliased symbol representing falsity (the underlying implementation for falsity is `nil`) |
|number   | Any integer, or floating point integer for example `1`, `19`, `3.14159265` |
|character| Any letter prepended with a backslash, for example `\a` represents the first letter of the english alphabet. |

### Cons Cells

The fundamental data type in Bunny is a pair of two things, also known as a cons cell. A first element (called `head`) and a second element (called `tail`) are all that make up a cons cell.

The syntax for a cons cell is `(cons <head> <tail>)`, for example the pair for two symbols, `a` and `b` is written as `(cons a b)`. The dotted-pair notation also represents cons cells, for example `(<head> . <tail>)` is equivalent to `(cons <head> <tail>)`.

### Lists

A list is an arbitrary length sequence of cons cells. The `tail` of each cons cell holds the next cons cell, or contains `nil` as a special symbol designating sequence termination. Any data structure formed with cons cells is a list.

The underlying representation for a list is a cons pair as described below:

```
;; All three of the forms below are equivalent.
(1 2 3)
(1 . (2 . (3 . ()))
(cons 1 (cons 2 (cons 3 nil)))
```

| **Examples** | **Semantic Meaning** |
|--------------|--------------------|
|`()`          | an empty list |
|`nil`        | an empty list, equivalent to `()` |
|`(a)`         | a list containing the symbol `a` |
|`(a ())`      | equivalent to `(a)`, the symbol `a` and the empty list `()` |
|`(a nil)`    | equivalent to `(a)`, the symbol `a` and empty list `nil` |
|`(a (b c))`   | two element list of the symbol `a` and the list `(b c)` |

List are always evaluated, but it's often useful to work with and manipulate them without evaluation. This is done using the concept of quoting. The following examples show some basic examples.

```
;; The following is evaluated
(+ 1 2)
=> 3

;; This is not evaluated
(quote (+ 1 2))
=> (+ 1 2)

;; Syntactic sugar form, equivalent to the above
'(+ 1 2)
=> (+ 1 2)
```

`head` and `tail` are also functions that can be applied to lists, for example, `(head '(1 2 3))` evaluates to `1` and `(tail '(1 2 3))` evaluates to `(2 3)`. It's possible to get the `nth` item using the function `nth`, e.g. `(nth 2 '(1 2 3)` evaluates to `3`.

Given an empty list `()`, `(head '())` is `()` and `(tail '())` is `()`. In other words, the `head` and `tail` of an empty list are `nil`.

### Variables

Variables can be defined globally and mutated, though mutation should be used sparingly. This interface is exposed to the user since practical applications often require variable mutation, for example for configuring runtime behavior based on some variables. The form for defining a global variable is `define` and `set!` for mutation. Note that the `!` signifies mutation, so even user defined functions that change state should by convention be suffixed with `!`.

```
(define foo "foo")  ; Defines a global variable foo and binds it to the value "foo".
(set! foo "bar")  ; Mutates the variable foo and binds it to the value "bar".
```

Variables can be lexically scoped using `let`, and then used within the scoped expression. The form used is `(let (<bindings>) (<expression>))`.

```
;; bind foo to the value 42 and return its square
(let ((foo 42))
  (* foo foo))
```

Multiple bindings can be used in a `let` form.

```
(let ((foo 1)
      (bar 2))
  (+ foo bar))
```

Note that the behavior of `let` evaluates each binding immediately and in-order, allowing dependent bindings.

```
(let ((foo 2)
      (bar (+ foo 1))  ; use the previous binding of foo and add 1 to it
  (* foo bar))
=> (* 2 3) => 6
```

### Functions

Bunny has two forms of functions, anonymous functions (`lambda` or `λ`) and named functions which are just syntactic sugar for a `λ` inside a `define` form.

Anonymous functions take the form `(λ (<arguments>) (<expression>))`. Below is an example that squares a number.

```
(λ (x) (* x x))
```

Since functions are values, and we use `define` to give names to values, we can use `define` and `lambda` to express a named function.

```
(define <function_name>
  (λ (<arguments>)
    (<expression>)))
```

For example, we can define the function `incr` that increments a given integer.

```
(define incr
  (λ (x)
    (+ x 1)))
```

And then invoke it:

```
(incr 1)
=> 2
```

Bunny should implement a special short-hand form for `define` as a more convenient way to define functions. The `define` form for named functions is is `(define (<function_name> <arguments>) (<expression>))`. The same `incr` function defined using the `define` short-hand form below.

```
(define (incr x)
  (+ x 1))
```

Functions should generally have documentation strings describing them, except in trivial cases or for internal helper methods.

### Conditionals

Basic conditional logic forms in Bunny are pretty similar to Scheme. Below are the forms conditionals can take with some examples.

`if` blocks take the form `(if (<condition>) (<true_expression>) (<false_expression>))`.

```
(if (< 1 0)
  "condition met"
  "condition failed")
=> "condition failed"
```

`when` is a macro that expands to `(if (<condition>) (<true_expression>) (nil))`. It is preferred when there's no `else` clause needed.

```
(when (< 1 2)
  "condition met")
=> "condition met"

(when (< 1 2)
  "condition met")
=> nil
```

`cond` blocks are a slightly more generic way to construct multiple conditions, they take the form `(cond (<conditional_0>) (<expression_0>) ... (<conditional_n>) (<expression_n>))`.

```
;; Assume x is bound or supplied by a function argument, this condition will return a string
;; based on the value of x.
(cond ((< x 10) "less than 10")
      ((< x 100) "less than 100")
      (else "no conditions met"))

;; If no result expression is given, the condition block will return the result of the conditional expression.
(cond ((< 1 10)))
=> true
```

`and` is simply the logical and. `or` is the logical or.

```
(and (< 1 10) (< 2 10))
=> true

(and (< 10 1) (< 2 10))
=> false

(or (< 1 10) (< 2 10))
=> true

(or (< 5 1) (< 2 3))
=> true

(or (< 5 1) (< 3 2))
=> false
```

`not` negates the boolean expression immediately following it.

```
(not (< 1 2))
=> false
```

`unless` is a macro preferred over `(if (not ..) .. ..)` and evaluates to an equivalent `if` block with the conditional expression negated.

```
(unless (even? 2)
  "number is not even"
  "number is even!")
=> "number is even!"

;; unless can be used without a second clause, this makes it an implicit nil
(unless (odd? 2) "number is not odd")
=> "number is not odd"

(unless (odd? 3) "number is not odd")
=> nil
```

### Namespaces

The default global namespace is `bunny`. When launching a REPL, the user will be in the `bunny` namespace. Unless explicitly specified, definitions will automatically be in the `bunny` namespace, and it's possible to override existing definitions. This namespace contains the core language, the standard library, and any globals.

- `(namespace <name>)` will either create a new namespace, or set the current namespace if `<name>` already exists.
- `(export [<definition_0> ... <definition_n>])` will mark definitions as exported, meaning they can be imported in other namespaces.
- `(import <name>)` will import any exported definitions from `<name>` into the current namespace.

Example:

```
;;; # foo

;; Create a new namespace foo, re-use if already exists.
(namespace foo)
(export [function
         magic-name])

(define (function name)
  (print name))

(define magic-name "Magic Foo")

;;; # bar
(namespace bar)

(foo/function foo/magic-name)  ; this is valid and will print "Magic Foo"

(import foo)
(function magic-name)  ; though importing does not require prefixing

;; Note that definitions from imports can be mutated. This will change the definition
;; globally, so other parts of code that reference magic-name will also be affected.
(set! magic-name "Changed")
(function magic-name)  ; this will now print "Changed"
```

Namespaces can also be nested, this should provide sufficient flexibility when writing larger pieces of software. Building on the example above for the namespace `foo`, we can add nested namespaces as shown below.

```
;;; foo.utils

(namespace foo.utils)
(export [utility-function])

(define (utility-function <args>)
  <body>)

;; In any other namespace, we can call foo.utils/utility-function explicitly
;; or we can import foo.utils and call utility-function.
(foo.utils/utility-function <args>)  ; explicitly address namespaced function
(import foo.utils)                   ; import exported functions into current namespace
(utility-function <args>)            ; call function as if it's in the same namespace
```

There's an obvious issue of definition names colliding, but the it would be preferable for tooling to throw an error during compilation about overriding existing definitions. For example, the following should warn about the re-definition.

```
(define + "foo")
```

```
Error: symbol + is already defined.
```

Functions (or any value) can be re-defined in a namespace without interfering with identically named definitions in other namespaces, but importing another namespace would override the definition in the current namespace. This can potentially be useful for monkey-patching, but should be considered risky practice. It's also possible to override definitions globally using `set!`, though this is discouraged and should be used sparingly.

### Comments

Comments in Bunny are specified with semi-colons (`;`) with three distinct types of comments.

* **Inline comments:** a single semi-colon `;` with two preceding whitespace characters are used for in-line comments. Used for very small comments, should be used sparingly.
  ```
  (+ 1 2)  ; Add one and two.
         ^^
      whitespace
  ```
* **Comment blocks:** two semi-colons `;;` which can span multiple lines. Should be used generously, especially around complex code blocks and should be aligned with the line directly after.
  ```
  ;; The code below adds one and two
  ;; using the + operator.
  (+ 1 2)
  ```
* **Documentation comment blocks:** three semi-colons `;;;` spanning multiple lines. These are used for documentation and parsed out by the documentation generator to create web docs for libraries. Markdown is supported.
  ```
  ;;; # my-math
  ;;; The my-math namespace provides a single function my-addition to add two integers.

  (namespace my-math)
  (export [my-addition])

  ;;; my-addition takes two arguments and sums them together.
  ;;;
  ;;; usage: `(my-addition 1 2)`
  (define (my-addition x y)
    (+ x y))
  ```

---

## Libraries (Carrots)

Bunny calls libraries carrots, as in "add the postgresql carrot to your dependencies". Creating and distributing carrots should be an integrated part of the `bunny` tooling, much like Ruby with it's incredible [bundler](https://bundler.io/) tool. The tooling should also handle documentation generation, similar to [rustdoc](https://doc.rust-lang.org/rustdoc/what-is-rustdoc.html).

### Creating a Library

Creating a new `carrot` should be as simple as:

```
$ bunny carrot new <carrot_name:required> --bin=cli  # example to add an optional binary executable entrypoint to the carrot
```

This should setup a directory `<carrot_name>` with the skeleton for a new carrot. Modify the `carrot_spec.bn` accordingly, and code away. Other projects can use this code now by adding `<carrot_name>` to its `carrots.bn` file and importing `<carrot_name>` in code. Below is what the directory structure will look like.

```
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

```
(namespace <carrot_name>)
(export [magic-number
         foo])

(define magic-number 42)

;;; Prints whatever argument is passed to the function.
(define (foo argument)
  (print argument))
```

### Adding a Dependency

All project or library dependencies should be added to the `carrots.bn` file either manually or by invoking the `bunny` command.

```
$ bunny carrot add <carrot_name:required> <carrot_version:optional>
```

This will automatically add the dependency to the `carrots.bn` file.

### Using a Library

Functions in `<carrot_name>.bn` can be used after importing, below is usage based on the above example for [Creating a Library](#creating-a-library).

```
(<carrot_name>/foo "rabbits are cool and stuff")
(<carrot_name>/magic-number)  ; This will evaluate to 42.

(import <carrot_name>)

(foo "this is much shorter!")
(magic-number)         ; Still evaluates to 42.
(set! magic-number 1)  ; Mutate the variable globally.
(magic-number)         ; Now evaluates to 1.
```

### Versioning

All `carrot`s should follow [Semantic Versioning](https://semver.org/). For example, the first stable release of Bunny will be `v1.0.0`.

### Distributing

`carrot`s are encouraged to be published and distributed through the [carrot registry](#carrot-registry), which is the default built into the dependency manager. It is also possible to distribute `carrot`s through [GitHub Releases](https://docs.github.com/en/github/administering-a-repository/about-releases).

---

### `bunny-up`

The `bunny-up` tool is the simplest way to get started with `bunny`, installing the tool is simple and allows installing switching between `bunny` versions. This is based almost entirely off the great [`rustup`](https://rustup.rs/) tool. Install the tool by running the command below on a unix-like system. The source-code for `bunny-up` can be found at [https://github.com/bunny-lang/bunny-up](https://github.com/bunny-lang/bunny-up).

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

All public `carrot`s are distributed to the centralized `carrot` registry at [https://carrots.bunny-lang.org](https://carrots.bunny-lang.org). Hosting a private `carrot` registry should be easily possible with standard webservers. `carrot`s are uniquely namespaced according to their `git` url, this gives us a uniqueness guarantee for free and discourages bizarre naming schemes to avoid collisions.

A project can declare multiple registry sources in its `carrots.bn` file, and `carrot`s will be searched through the listed registries in order to resolve the package.

### Documentation Generator

TODO...

### Centralized Documentation

Tightly integrated with the Centralized Carrot Registry, [https://docs.bunny-lang.org](https://docs.bunny-lang.org) offers a single source of documentation for the core language, the standard library, and user published `carrot`s.

---

## Conventions and Style Guide

Bunny encourages a set of conventions to make it both more approachable and simplifies tooling by being able to make certain assumptions. The ideal would be to have tooling make it hard to break these conventions, and provide enough automation to make the defaults easy to get started with.

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

All `bunny` code should avoid ambiguity whenever possible. This applies to file, variable, function, and other names. Explicit over implicit. Full names over abbreviations.

### Document Carrot Interfaces

Each `carrot` should document its public interface (i.e. all definitions that are exported) using the documentation comment syntax above definitions. When distributing `carrot`s, the documentation generator can publish the documentation in the centralized `carrot` repository in `Markdown` and provide a uniform way to browse documentation.