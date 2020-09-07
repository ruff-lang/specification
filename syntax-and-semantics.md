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

```
(1 2 3) => (cons 1 (cons 2 (cons 3 nil))) => (1 . (2 . (3 . ()))
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

```lisp
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

Variables can be defined globally and mutated, though mutation should be used sparingly. This interface is exposed to the user since practical applications often require variable mutation, for example for configuring runtime behavior based on some variables. The form for defining a global variable is `def` and `mut` for mutation.

```clojure
(def foo "foo")  ; Defines a global variable foo and binds it to the value "foo".
(mut foo "bar")  ; Mutates the variable foo and binds it to the value "bar".
```

Variables can be lexically scoped using `let`, and then used within the scoped expression. The form used is `(let (<bindings>) (<expression>))`.

```lisp
;; bind foo to the value 42 and return its square
(let ((foo 42))
  (* foo foo))
```

Multiple bindings can be used in a `let` form.

```lisp
(let ((foo 1)
      (bar 2))
  (+ foo bar))
```

Note that the behavior of `let` evaluates each binding immediately and in-order, allowing dependent bindings.

```lisp
(let ((foo 2)
      (bar (+ foo 1))  ; use the previous binding of foo and add 1 to it
  (* foo bar))
=> (* 2 3) => 6
```

## Functions

Bunny has two forms of functions, anonymous functions (`lambda` or `λ`) and named functions (`defun`). Named functions are just syntactic sugar for a `λ` inside a `def` form.

Anonymous functions take the form `(λ (<arguments>) (<expression>))`. Below is an example that squares a number.

```clojure
(λ (x) (* x x))
```

Since functions are values, and we use `define` to give names to values, we can use `define` and `lambda` to express a named function.

```clojure
(def <function_name>
  (λ (<arguments>)
    (<expression>)))
```

For example, we can define the function `incr` that increments a given integer.

```clojure
(def incr
  (λ (x)
    (+ x 1)))
```

And then invoke it:

```lisp
(incr 1) => 2
```

Bunny should implement a special form `defn` as a more convenient way to define functions. The `defn` form is `(defn <function_name> (<arguments>) <documentation_string:optional> (<expression>))`. The same `incr` function defined using `defn` below.

```clojure
(defn incr (x)
  "Increment the given integer by one."
  (+ x 1))
```

Functions should generally have documentation strings describing them, except in trivial cases or for internal helper methods.

## Conditionals

Basic conditional logic forms in Bunny are pretty similar to Scheme. Below are the forms conditionals can take with some examples.

`if` blocks take the form `(if (<condition>) (<true_expression>) (<false_expression>))`.

```lisp
(if (< 1 0)
  "condition met"
  "condition failed")
=> "condition failed"
```

`when` is a macro that expands to `(if (<condition>) (<true_expression>) (nil))`. It is preferred when there's no `else` clause needed.

```lisp
(when (< 1 2)
  "condition met")
=> "condition met"

(when (< 1 2)
  "condition met")
=> nil
```

`cond` blocks are a slightly more generic way to construct multiple conditions, they take the form `(cond (<conditional_0>) (<expression_0>) ... (<conditional_n>) (<expression_n>))`.

```lisp
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

```lisp
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

```lisp
(not (< 1 2))
=> false
```

`unless` is a macro preferred over `(if (not ..) .. ..)` and evaluates to an equivalent `if` block with the conditional expression negated.

```lisp
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

To avoid overriding definitions and giving some structure to our code, we can organize code into namespaces.

```clojure
;; Create a new namespace my-math, reuse if already exists.
(ns my-math
  ;; We use the math namespace and alias it to m.
  (use (math :as m)))
```

## Comments

Comments in Bunny are specified with semi-colons (`;`) with three distinct types of comments.

* **Inline comments:** a single semi-colon `;` with two preceding whitespace characters are used for in-line comments. Used for very small comments, should be used sparingly.
  ```lisp
  (+ 1 2)  ; Add one and two.
         ^^
      whitespace
  ```
* **Comment blocks:** two semi-colons `;;` which can span multiple lines. Should be used generously, especially around complex code blocks and should be aligned with the line directly after.
  ```lisp
  ;; The code below adds one and two
  ;; using the + operator.
  (+ 1 2)
  ```
* **Top-level comment blocks:** same as comment blocks but limited to the top of a file with a brief summary of the code in that file.
  ```clojure
  ;; The my-math namespace provides a single function my-addition to add two integers.

  (ns my-math)

  (defn my-addition (x y)
    "Takes two integer arguments and returns the sum of them."
    (+ x y))
  ```