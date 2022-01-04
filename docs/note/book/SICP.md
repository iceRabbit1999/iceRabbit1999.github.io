***Structure and Interpretation of Computer Programs***



# 1 Building Abstractions with Procedures

1. we conjure the spirits of the computer with our spells
2. A computational process is indeed much like a sorcerer's idea of a spirit

**Programming in Lisp**

1.  The most significant of these features is the fact that Lisp descriptions of processes, called *procedures*, can themselves be represented and manipulated as Lisp data

## 1.1 The Elements of Programming

1. we should pay particular attention to the means that the language provides for combining simple ideas to form more complex ideas. Every powerful language has three mechanisms for accomplishing this:

- primitive expressions,which represent the simplest entities the language is concerned with,
- means of combination, by which compound elements are built from simpler ones, and
- means of abstraction, by which compound elements can be named and manipulated as units

2. In programming, we deal with two kinds of elements: procedures and data

### 1.1.1 Expressions

```scheme
(+ 137 349)
486
(- 1000 334)
666
(* 5 99)
495
(/ 10 5)
2
(+ 2.7 10)
12.7

nested
(+ (* 3 5) (- 10 6))
19
```

Even with complex expressions, the interpreter always operates in the same basic cycle: It reads an expression from the terminal, evaluates the expression, and prints the result. This mode of operation is often expressed by saying that the interpreter runs in a ***read-eval-print loop***.

### 1.1.2 Naming and the Environment

```scheme
define size 2

size
2
```

The interpreter makes this step-by-step program construction particularly convenient because name-object associations can be created incrementally in successive interactions

### 1.1.3 Evaluating Combinations

1. isolate issues about thinking procedurally,to evaluate a combination,do the following
   1. Evaluate the subexpressions of the combination.
   2. Apply the procedure that is the value of the leftmost subexpression (the operator) to the arguments that are the values of the other subexpressions (the operands). (将作为最左边的子表达式（运算符）的值的程序应用于作为其他子表达式（操作数）的值的参数)