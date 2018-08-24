# Building Abstractions with Procedures

This book uses the programming language **LISP (LISt Processing)** and, more specifically, a dialect of LISP called **Scheme**.

**Helpful Links:**
* <a href="http://www.gnu.org/software/mit-scheme/documentation/mit-scheme-user/Unix-Installation.html#Unix-Installation">Installation Instructions</a>
* <a href="http://www.gnu.org/software/mit-scheme">Download page</a>
* <a href="http://www.gnu.org/software/mit-scheme/documentation/mit-scheme-user/index.html">Using Scheme Guide</a>

To use the scheme CLI, run `$ mit-scheme`, and to exit use `^` (ctrl) + `D`.

To run a file we can do `$ mit-scheme --load *filename*`.


## 1.1 The Elements of Programming

When we describe a language, we should pay attention to the means that the language provides for combining simple ideas to form more complex ones. Every powerful language has 3 mechanisms for accomplishing this:

* **primitive expressions** - represent the simplest entities the language is concerned with.
* **means of combination** - by which compound elements are built from simpler ones.
* **means of abstraction** - by which compound elements can be named and manipulated as units.

In programming we deal with 2 kinds of elements:

1. Procedures - Descriptions of the rules for manipulating data.
2. Data - "Stuff" we want to manipulate

*In this chapter we will deal only with simple numerical data so that we can focus on the rules for building procedures. In later chapters we will see that these same rules allow us to build procedures to manipulate compound data as well.*


### 1.1.1 Expressions

Let's view how a Scheme CLI will handle us doing basic numerical things.

``` scheme
486

; 486
```

Expressions representing numbers may be combined with an expression representing a primitive procedure (such as `+` or `*`) to form a compound expression that represents the application of the procedure to those numbers.

``` scheme
(+ 137 349)
; 486

(-1000 334)
; 666

(* 5 99)
; 495

(/ 10 5)
; 2

(+ 2.7 10)
; 12.7
```

Expressions such as these, formed by *delimiting a list of expressions within parentheses in order to denote procedure application*, are called **combinations**. The leftmost element in the list is called the **operator**, and the other elements are called **operands**.

The convention of placing the operator to the left of the operand is known as **prefix notation**, and it may be somewhat confusing at first because it departs significantly from the customary mathematical convention. Prefix notation does have several advantages. One is that it can accommodate procedures that may take a larger number of arguments:

``` scheme
(+ 21 35 12 7)
; 74

(* 25 4 12)
; 1200
```

It won't be confusing because the operator is always the leftmost element and the entire combination is delimited by the parentheses.

A second advantage is that it extends in a straight-forward way to allow combinations to be nested. Combinations whose elements are themselves combinations:

``` scheme
(+ (* 3 5) (- 10 6))
; (+ 15 4)
; 19
```

There's no limit to the depty of such nesting. It's just humans who can get confused.

``` scheme
(+ (* 3 (+ (* 2 4) (+ 3 5))) (+ (- 10 7) 6))
; 57
```

We can help ourselves by writing the expression like so:

``` scheme
(+ (* 3
      (+ (* 2 4)
         (+ 3 5)))
   (+ (- 10 7)
      6))
```

This follows a formatting convention known as **pretty-printing**, in which each long combination is writting so that the operands are aligned vertically.

The interpreter always operates in the same basic cycle: It reads an expression, evaluates the expression, and prints the result. This mode of operation is often expressed by saying that the interpreter runs in a **read-eval-print loop** (REPL). *Observe in particular that it is not necessary to explicitly instruct the interpreter to print the value of the expression.*


### 1.1.2 Naming and the Environment

A critical aspect of a programming language is the means it provides for using names to refer to computational objects. We say that the name identifies a **variable** whose **value** is the object.

In Scheme, we name things with `define`.

``` scheme
(define size 2)
```

This causes the interpreter to associate the value 2 with the name `size`. Once the name `size` has been associated with the number 2, we can refer to the value 2 by name:

``` scheme
size
; 2

(* 5 size)
; 10
```

Further examples of `define`:

``` scheme
(define pi 3.14159)

(define radius 10)

(* pi (* radius radius))
; 314.159

(define circumference (* 2 pi radius))

circumference
; 62.8318
```

`Define` is our langauge's simplest means of abstration, for it allows us to use simple names to refer to the results of compound operations, such as the `circumference` computed above. Complex programs are constructed by building, step by step, computational objects of increasing complexity.

It should be clear that the possibility of associating values with symbols and later retrieving them means that the interpreter must maintain some sort of memory that keeps track of the name-object pairs. This memory is called the **environment** (more precisely the **global environment**, since we will later see that a computation may involve a number of different environments).


### 1.1.3 Evaluating Combinations

The interpreter is itself following a procedure to evaluate combinations:

1. Evaluate the subexpressions of the combination.
2. Apply the procedure that is the value of the leftmost subexpression (the operator) to the arguments that are the values of the other subexpressions (the operands).

The evaluation rules are **recursive** in nature because in order to accomplish the evaluation process for a combinatino we must first perform the evaluation process on each element of the combination.

``` scheme
(* (+ 2 (* 4 6))
   (+ 3 5 7))
```

Evaluating the above requires the evaluation rule be applied to four different combniations. We can view this process by representing the combination in the form of a tree in Figure 1.1. Each combination is represented by a node with branches corresponding to the operator and the operands of the combination stemming from it. 

<img src="images/figure1-1.gif?raw=true" alt="Figure 1.1:  Tree representation, showing the value of each subcombination." style="margin: 0 auto;">

Viewing evaluation in terms of the tree, we can imagine that the values of the operands percolate upward, starting from the terminal nodes (nodes with no branches) and then combining at higher and higher levels.

Recursion is a powerful technique for handling heirarchical, tree-like objects. "Percolate values upward" form of the evaluation rule is an example of a general kind of process known as **tree accumulation**.

The repeated use of the first step brings us to the point where we are only evaluating primitive expressions such as numerals, built-in operators, or other names. We take care of the primitive cases by stipulating that

* the values of numerals are the numbers that they name,
* the values of built-in operators are the machine instruction sequences that carry out the corresponding operations, and
* the values of other names are the objects associated with those names in the environment.

In Lisp, it is meaningless to speak of the value of an expression such as `(+ x 1)` without specifying any information about the environment that would provide a meaning for the symbol `x` (or even `+`).

The evaluation rule does not handle definitions. `(define x 3)` does not apply `define` to 2 arguments, since the purpose of `define` is to associate `x` with a value. (`(define x 3)` is not a combination.)

Exceptions to the general evaluation rule are called **special forms**. `Define` is the only example of a special form that we have seen so far. Each special form has its own evaluation rule. The various expressions (each with its associated evaluation rule) constitute the syntax of the programming langauge.


### 1.1.4 Compound Procedures

We have identified in Lisp some of the elements that must appear in any powerful programming language:

* Numbers and arithmetic operators are primitive data and procedures.
* Nesting of combinations provides a means of combining operations.
* Definitions that associate names with values provide a limited means of abstraction.

Now we will learn about **procedure definitions**, a much more powerful abstraction technique by which a compound operation can be given a name and then referred to as a unit.

Let's try to express the idea of "squaring". "To square something, multiply it by itself." This would be expressed in the language as:

``` scheme
(define (square x) (* x x))
```

We can understand this in the following way:

![1.1.4](images/square-1.1.4.gif?raw=true "Square")

We have here a **compound procedure**, which has been given he name `square`. Basically, it's a function in most other languages. It's general form looks like this:

```
(define (<name> <formal parameters>) <body>)
```

The `<name>` is a symbol to be associated with the procedure definition in the environment. The `<formal parameters>` are the names used within the body of the procedure to refer to the corresponding arguments of the procedure. The `<body>` is an expression that will yield the value of the procedure application with the formal parameters are replaced by the actual arguments to which the procedure is applied.

Having defined `square`, we can now use it:

``` scheme
(square 21)
; 441

(square (+ 2 5))
; 49

(square (square 3))
; 81
```

We can also use square as a building block in defining other procedues. For example, `x^2 + y^2` can be expressed as

``` scheme
(+ (square x) (square y))
```

We can define a procedure `sum-of-square` that, given any two numbers as arguments, produces the sum of their squares:

``` scheme
(define (sum-of-square x y)
    (+ (square x) (square y)))

(sum-of-square 3 4)
; 25
```

Now we can use `sum-of-squares` as a building block in constructing further procedures:

``` scheme
(define (f a)
    (sum-of-squares (+ a 1) (* a 2)))

(f 5)
; 136
```

Compound procedures are used in exactly the same way as primitive procedures. Indeed, one could not tell by looking at the definition of `sum-of-squares` given above whether `square` was built into the interpreter, like `+` and `*`, or defined as a compound procedure.


### 1.1.5 The Substitution Model for Procedure Application




### 1.1.6 Conditional Expressions and Predicates

### 1.1.7 Example: Square Roots by Newton's Method

### 1.1.8 Procedures as Black-Box Abstractions