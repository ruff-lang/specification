<!---
  This Source Code Form is subject to the terms of the Mozilla Public
  License, v. 2.0. If a copy of the MPL was not distributed with this
  file, You can obtain one at http://mozilla.org/MPL/2.0/.
-->

### File Format

Bunny source files must be [UTF-8](https://en.wikipedia.org/wiki/UTF-8) with the `.bn` file extension.

### Atoms

An `atom` is a singular piece of data.

|**Atom** |**Semantic Meaning**|
|---------|--------------------|
|symbol   | A symbol is a word or an operator, for example `foo` is a symbol and so is `+` |
|keyword  | A keyword is like a symbol that evaluates to itself, for example `:foo` is a keyword |
|boolean  | `true` is a symbol representing the truthy boolean. `false` is an aliased symbol representing falsity (the underlying implementation for falsity is `nil`) |
|number   | Any integer, or floating point integer for example `1`, `19`, `3.14159265` |
|character| Any letter prepended with a backslash, for example `\a` represents the first letter of the english alphabet. |

### Cons Cell (Pairs)

Fundamental data type in Bunny. A cons cell is a pair of two things. The first element is called the `head` and the second element is called the `tail` and is a pointer to another cons cell. `nil` terminates a sequence of cons cells.

```
+------+------+
| head | tail |
+------+------+
   |      |
   |      |__ this holds a pointer to another pair or nil
   |
   |__ this holds an atom or a pair
```

Syntax to build cons cells is `(cons <head> <tail>)`. Dotted-pair notation is syntactic sugar, `(<head> . <tail>)`.

### Lists

Lists are arbitrarily long sequences of cons cells. Data structures formed with cons cells are lists. All three forms below are equivalent.

```
(1 2 3)
(1 . (2 . (3 . nil)))
(cons 1 (cons 2 (cons 3 nil)))
```

`()` is the empty list and is equivalent to `nil`.

Lists are always evaluated unless explicitly quoted. Quoting stops evaluation. Lists can be quoted with `(quote <list>)` or the syntactic sugared form, `'(<list>)`.

```
(+ 1 2)
=> 3

(quote (+ 1 2))
=> (+ 1 2)

'(+ 1 2)
=> (+ 1 2)
```

Inside a quoted form, we can resume evaluation selectively. This is called unquoting. Tilde (`~`) is the suntax for unquoting.

```
(quote (+ 1 (unquote (+ 2 3))))
=> (+ 1 5)

# equivalent, using syntactic sugar
'(+ ~(+ 2 3))
=> (+ 1 5)
```

Forms can be unquoted (`unquote-splice`) into the position in the quoted template. This is useful for when you want to flatten a list. The at-sign (`@`) is the syntax for unquote-splice. For example:

```
'(1 2 ~(list 3 4))
=> (1 2 (3 4))

(quote (1 2 (unquote-splice (list 3 4))))
=> (1 2 3 4)

# equivalent, using syntactic sugar
'(1 2 @(list 3 4))
=> (1 2 3 4)
```

`head` is a function returning the head of a list.

```
(head '(1 2 3))
=> 1
```

`tail` is a function returning the tail.

```
(tail '(1 2 3))
=> (2 3)
```

### Comments

Comments in Bunny are specified with a single hashtag, `#`.

```
# this is a comment

# this is a comment that
# spans multiple lines
```

In-line comments should have two whitespaces preceding.

```
(+ 1 2)  # add one and two
       ^^
   whitespace
```

### Variables

Values can be defined globally with `(define <name> <body>)`. Mutation is possible with `(set! <name> <value>)` but discouraged in actual logic.

```
(define foo 42)
(println foo)
=> 42

(set! foo (+ foo foo))
(println foo)
=> 84
```

Variables can be lexically scoped using `let` and then used within the scoped expression.

```
(let ((foo 42))
  (+ foo foo))
=> 84
```

Multiple bindings can be used.

```
(let ((foo 1) (bar 2))
  (+ foo bar))
=> 3
```

`let` evaluates each binding immediately and in-order, allowing for dependent bindings.

```
(let ((foo 2)
      (bar (* foo foo)))
  (+ foo bar))
=> 6
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

A special short-hand form `defun` is a convenient way to define named functions with arguments. `(defun <function_name> (<arguments>) (<expression>))`. The same `incr` function defined using the `defun` short-hand form below.

```
(defun incr (x)
  (+ x 1))
```

### Conditionals

Basic conditional logic forms in Bunny are pretty similar to Scheme. Below are the forms conditionals can take with some examples.

`if` blocks take the form `(if (<condition>) (<true_expression>) (<false_expression>))`.

```
(if (< 1 0)
  (println "condition met")
  (println "condition failed"))
=> "condition failed"
```

`when` is a macro that expands to `(if (<condition>) (<true_expression>) (nil))`. It is preferred when there's no `else` clause needed.

```
(when (< 1 2) (println "condition met"))
=> "condition met"

(when (< 1 2) (println "condition met"))
=> nil
```

`cond` blocks are a slightly more generic way to construct multiple conditions, they take the form `(cond (<conditional_0>) (<expression_0>) ... (<conditional_n>) (<expression_n>))`.

```
# Assume x is bound or supplied by a function argument, this
# condition will return a string based on the value of x.
(cond ((< x 10) "less than 10")
      ((< x 100) "less than 100")
      (else "no conditions met"))

# If no result expression is given, the condition block will
# return the result of the conditional expression.
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
  (println "not even")
  (println "even"))
=> "even"

# unless can be used without a second clause making it an implicit nil
(unless (odd? 2) (println "not odd"))
=> "not odd"

(unless (odd? 3) (println "not odd"))
=> nil
```

### Errors

TODO...

### Concurrency

Lightweight threads are used for concurrency, also known as coroutines, green threads, and fibers. The runtime handles all concurrency tasks and exposes a simple interface with fibers. Messages can be shared across fibers using queues.

You can start a new concurrent task with `fiber`.

```
(fiber
  (while true
    (sleep 10)
    (println "brr")))
```

Fibers can be exited out with `(done)`. Every fiber implicitly has a reference to its parent fiber, and as such can invoke the `parent-done?` method to detect if the parent exited out.

```
(fiber
  (while true
    (when (parent-done?) (done))
	(println "brr")))
```

Queues can be created with `queue`.

```
(define some-queue (queue))
```

Things can be added to a queue with `put`.

```
(put some-queue "foo")
```

And things can be taken off a queue with `take`, which blocks until the queue has something on it.

```
(let ((msg (take some-queue)))
  (println (format "got message: %s" msg))))
```

We can put an error on the queue as a signal to a fiber to terminate. Here's an example of a fiber that prints messages received til it sees an error.

```
(let ((msgs (queue)))
  (fiber 
    (while true
      (let ((msg (take msgs)))
        (when (error? msg) (done))
        (println (format "received: %s" msg))))))
  (put q "hello")
  (put q "hello again!")
  (put q (error "signaling an error")))

=> "received: hello"
   "received: hello again!"
   nil
```

Note that using `define` will create a queue in the global scope and will not be garbage collected. Using `let` will clean up the queue resources once that bound variable is no longer referenced.

### Modules

Modules provide a way of organizing and grouping code together. They also provide a convenient way to distinguish between code meant to be shared or exposed (in the case of a library) and code meant for internal use only.

A module can be defined to only explicitly export a list of public functions. Otherwise, all definitions inside the module will be implicitly exported. We can define a new module using `defmodule`, and tell the language where we're implementing that module with `module`. Note that both the module definition and the implement directive can be in the same file.

```
# mod.bn

(defmodule Utilities
  (export (print-uppercase
           print-lowercase)))
```

```
# utilities.bn

(module Utilities)

(defun print-uppercase (arg)
  (let ((uppercase (String.upper arg)))
    (println uppercase)))
```

In another file this module can be used.

```
(Utilities.print-uppercase "hello")
=> "HELLO"
```

To avoid having to use the dot notation, the module can be imported with `use`.

```
(use Utilities)

(print-uppercase "hello")
=> "HELLO"
```

If a symbol is already defined leading to a name conflict, `(use <module>)` will throw an error at build time.

Alternatively, a file containing definitions _not_ in a module can be imported for use. Define in one file:

```
# helpers.bn

(defun roll-dice ()
  (Random.pick '(1 2 3 4 5 6))
```

and import in another using relative path:

```
# application.bn

(import "helpers.bn")

(roll-dice)
=> 6
```

If `application.bn` already defines a symbol in `helpers.bn`, `import` will throw an error at build time.

### Macros

TODO...
