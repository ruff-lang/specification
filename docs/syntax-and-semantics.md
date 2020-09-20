# Syntax and Semantics

## Table of Contents

* [Atoms](#atoms)
* [Cons Cells](#cons-cells)
* [Lists](#lists)
* [Variables](#variables)
* [Functions](#functions)
* [Conditionals](#conditionals)
* [Namespaces](#namespaces)
* [Comments](#comments)

## Atoms

An `atom` is any singlular piece of data that is not a pair. Bunny has a couple of different kinds of atoms.

|**Atom** |**Semantic Meaning**|
|---------|--------------------|
|symbol   | A symbol is a word or an operator, for example `foo` is a symbol and so is `+` |
|keyword  | A keyword is like a symbol that is exported and evaluates to itself, for example `:foo` is a keyword |
|boolean  | `true` is a symbol representing the truthy boolean. `false` is an aliased symbol representing falsity (the underlying implementation for falsity is `nil`) |
|number   | Any integer, or floating point integer for example `1`, `19`, `3.14159265` |
|character| Any letter prepended with a backslash, for example `\a` represents the first letter of the english alphabet. |

## Cons Cells

The fundamental data type in Bunny is a pair of two things, also known as a cons cell. A first element (called `head`) and a second element (called `tail`) are all that make up a cons cell.

The syntax for a cons cell is `(cons <head> <tail>)`, for example the pair for two symbols, `a` and `b` is written as `(cons a b)`. The dotted-pair notation also represents cons cells, for example `(<head> . <tail>)` is equivalent to `(cons <head> <tail>)`.

## Lists

A list is an arbitrary length sequence of cons cells. The `tail` of each cons cell holds the next cons cell, or contains `nil` as a special symbol designating sequence termination. Any data structure formed with cons cells is a list.

The underlying representation for a list is a cons pair as described below:

```scheme
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

```scheme
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

## Variables

Variables can be defined globally and mutated, though mutation should be used sparingly. This interface is exposed to the user since practical applications often require variable mutation, for example for configuring runtime behavior based on some variables. The form for defining a global variable is `define` and `set!` for mutation. Note that the `!` signifies mutation, so even user defined functions that change state should by convention be suffixed with `!`.

```scheme
(define foo "foo")  ; Defines a global variable foo and binds it to the value "foo".
(set! foo "bar")  ; Mutates the variable foo and binds it to the value "bar".
```

Variables can be lexically scoped using `let`, and then used within the scoped expression. The form used is `(let (<bindings>) (<expression>))`.

```scheme
;; bind foo to the value 42 and return its square
(let ((foo 42))
  (* foo foo))
```

Multiple bindings can be used in a `let` form.

```scheme
(let ((foo 1)
      (bar 2))
  (+ foo bar))
```

Note that the behavior of `let` evaluates each binding immediately and in-order, allowing dependent bindings.

```scheme
(let ((foo 2)
      (bar (+ foo 1))  ; use the previous binding of foo and add 1 to it
  (* foo bar))
=> (* 2 3) => 6
```

## Functions

Bunny has two forms of functions, anonymous functions (`lambda` or `λ`) and named functions which are just syntactic sugar for a `λ` inside a `define` form.

Anonymous functions take the form `(λ (<arguments>) (<expression>))`. Below is an example that squares a number.

```scheme
(λ (x) (* x x))
```

Since functions are values, and we use `define` to give names to values, we can use `define` and `lambda` to express a named function.

```scheme
(define <function_name>
  (λ (<arguments>)
    (<expression>)))
```

For example, we can define the function `incr` that increments a given integer.

```scheme
(define incr
  (λ (x)
    (+ x 1)))
```

And then invoke it:

```scheme
(incr 1)
=> 2
```

Bunny should implement a special short-hand form for `define` as a more convenient way to define functions. The `define` form for named functions is is `(define (<function_name> <arguments>) (<expression>))`. The same `incr` function defined using the `define` short-hand form below.

```scheme
(define (incr x)
  (+ x 1))
```

Functions should generally have documentation strings describing them, except in trivial cases or for internal helper methods.

## Conditionals

Basic conditional logic forms in Bunny are pretty similar to Scheme. Below are the forms conditionals can take with some examples.

`if` blocks take the form `(if (<condition>) (<true_expression>) (<false_expression>))`.

```scheme
(if (< 1 0)
  "condition met"
  "condition failed")
=> "condition failed"
```

`when` is a macro that expands to `(if (<condition>) (<true_expression>) (nil))`. It is preferred when there's no `else` clause needed.

```scheme
(when (< 1 2)
  "condition met")
=> "condition met"

(when (< 1 2)
  "condition met")
=> nil
```

`cond` blocks are a slightly more generic way to construct multiple conditions, they take the form `(cond (<conditional_0>) (<expression_0>) ... (<conditional_n>) (<expression_n>))`.

```scheme
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

```scheme
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

```scheme
(not (< 1 2))
=> false
```

`unless` is a macro preferred over `(if (not ..) .. ..)` and evaluates to an equivalent `if` block with the conditional expression negated.

```scheme
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

## Namespaces

The default global namespace is `bunny`. When launching a REPL, the user will be in the `bunny` namespace. Unless explicitly specified, definitions will automatically be in the `bunny` namespace, and it's possible to override existing definitions. This namespace contains the core language, the standard library, and any globals.

- `(namespace <name>)` will either create a new namespace, or set the current namespace if `<name>` already exists.
- `(export [<definition_0> ... <definition_n>])` will mark definitions as exported, meaning they can be imported in other namespaces.
- `(import <name>)` will import any exported definitions from `<name>` into the current namespace.

Example:

``` scheme
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

``` scheme
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

``` scheme
(define + "foo")
```

```
Error: symbol + is already defined.
```

Functions (or any value) can be re-defined in a namespace without interfering with identically named definitions in other namespaces, but importing another namespace would override the definition in the current namespace. This can potentially be useful for monkey-patching, but should be considered risky practice. It's also possible to override definitions globally using `set!`, though this is discouraged and should be used sparingly.

## Comments

Comments in Bunny are specified with semi-colons (`;`) with three distinct types of comments.

* **Inline comments:** a single semi-colon `;` with two preceding whitespace characters are used for in-line comments. Used for very small comments, should be used sparingly.
  ```scheme
  (+ 1 2)  ; Add one and two.
         ^^
      whitespace
  ```
* **Comment blocks:** two semi-colons `;;` which can span multiple lines. Should be used generously, especially around complex code blocks and should be aligned with the line directly after.
  ```scheme
  ;; The code below adds one and two
  ;; using the + operator.
  (+ 1 2)
  ```
* **Documentation comment blocks:** three semi-colons `;;;` spanning multiple lines. These are used for documentation and parsed out by the documentation generator to create web docs for libraries. Markdown is supported.
  ```scheme
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
