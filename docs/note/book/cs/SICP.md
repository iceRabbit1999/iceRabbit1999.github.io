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
   2. Apply the procedure that is the value of the leftmost subexpression (the operator) to the arguments that are the values of the other subexpressions (the operands). (??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????)
2. the evaluation rule is *recursive* in nature; that is, it includes, as one of its steps, the need to invoke the rule itself
3. The various kinds of expressions (each with its associated evaluation rule) constitute the syntax of the programming language
   1. for Lisp,the evaluation rule for expressions can be described by a simple general rule together with specialized rules for a small number of special forms

### 1.1.4 Compound Procedures

```scheme
#  how to express the idea of squaring
(define (square x) (* x x))

(square 21)
441

```

### 1.1.5 The Substitution Model for Procedure Application

1.  For compound procedures, the application process is as follows:
   1. To apply a compound procedure to arguments, evaluate the body of the procedure with each formal parameter replaced by the corresponding argument
2. The purpose of the substitution is to help us think about procedure application, not to provide a description of how the interpreter really works
3. The substitution model is only the first of these models -- a way to get started thinking formally about the evaluation process.

**Applicative order versus normal order**

1. normal order???fully expand and then reduce????????????????????????????????????????????????????????????(+ *??????),???????????????????????????????????????
2. applicative order: evaluate the arguments and then apply??????subcombination?????????????????????????????????
3. Lisp uses applicative-order evaluation, partly because of the additional efficiency obtained from avoiding multiple evaluations of expressions such as those illustrated with `(+ 5 1)` and `(* 5 2)` above and, more significantly, because normal-order evaluation becomes much more complicated to deal with when we leave the realm of procedures that can be modeled by substitution
4. for procedure applications that can be modeled using substitution???normal-order and applicative-order evaluation produce the same value

### 1.1.6 Conditional Expressions and Predicates

```scheme
(define (abs x)
  (cond ((> x 0) x)
        ((= x 0) 0)
        ((< x 0) (- x))))
# If none of the <p>'s is found to be true, the value of the cond is undefined
(define (abs x)
  (cond ((< x 0) (- x))
        (else x)))
(define (abs x)
  (if (< x 0)
      (- x)
      x))
#and
(and <e1> ... <en>)
#or
(or <e1> ... <en>)
#not
(not <e>)
```

### 1.1.7 Example:Square Roots by Newton's Method

1. The contrast between function and procedure is a reflection of the general distinction between describing properties of things and describing how to do things
   1. In mathematics we are usually concerned with declarative (what is) descriptions, whereas in computer science we are usually concerned with imperative (how to) descriptions

```scheme
(define (sqrt-iter guess x)
  (if (good-enough? guess x)
      guess
      (sqrt-iter (improve guess x)
                 x)))

(define (improve guess x)
  (average guess (/ x guess)))

(define (average x y)
  (/ (+ x y) 2))

(define (good-enough? guess x)
  (< (abs (- (square guess) x)) 0.001))

(define (sqrt x)
  (sqrt-iter 1.0 x))

(sqrt 9)
3.00009155413138
```

### 1.1.8 Procedures as Black-Box Abstractions

1. *procedural abstraction*??? We are not at that moment concerned with *how* the procedure computes its result, only with the fact that it computes the square. The details of how the square is computed can be suppressed, to be considered at a later time
   1. A user should not need to know how the procedure is implemented in order to use it
2. Local names:???????????????????????????????????????????????????????????????
   1. Internal definitions and block structure???????????????????????????????????????

## 1.2 Procedures and the Processes They Generate

1. The ability to visualize the consequences of the actions under consideration is crucial to becoming an expert programmer
2. A procedure is a pattern for the local evolution of a computational process

### 1.2.1 Linear Recursion and Iteration

1. ***recursive process***: 
   1. a shape of expansion followed by contraction
   2. characterized by a chain of deferred operations
   3. Carrying out this process requires that the interpreter keep track of the operations to be performed later on
   4. linear recursive:
      1. the length of the chain of deferred multiplications, and hence the amount of information needed to keep track of it, grows linearly with n
2. Iteration
   1.  In general, an iterative process is one whose state can be summarized by a fixed number of *state variables*, together with a fixed rule that describes how the state variables should be updated as the process moves from state to state and an (optional) end test that specifies conditions under which the process should terminate.
3. The contrast between the two processes
   1. In the iterative case, the program variables provide a complete description of the state of the process at any point
   2. Not so with the recursive process. In this case there is some additional ''hidden'' information, maintained by the interpreter and not contained in the program variables, which indicates ``where the process is'' in negotiating the chain of deferred operations
4. tail-recursive:
   1. we can say that a function is tail recursive if the final result of the recursive call ??? in this case 24 ??? is also the final result of the function itself, and that is why it???s called ???tail??? recursion
   2. because the final function call (the tail-end call) actually holds the final result

### 1.2.2 Tree Recursion

not important

### 1.2.3 Orders of Growth

### 1.2.4 Exponentiation

### 1.2.5 Greatest Common Divisors

### 1.2.6 Example:Testing for Primality

## 1.3 Formulating Abstractions with Higher-Order Procedures

1. One of the things we should demand from a powerful programming language is the ability to build abstractions by assigning names to common patterns and then to work in terms of the abstractions directly
2. Procedures that manipulate procedures are called *higher-order procedures*

### 1.3.1 Procedures as Arguments

```scheme
(define (identity x) x)

(define (sum-integers a b)
  (sum identity a inc b))

(sum-integers 1 10)
55
```

### 1.3.2 Constructing Procedures Using Lambda

1. In general, `lambda` is used to create procedures in the same way as `define`, except that no name is specified for the procedure(anonymous procedure/????????????)
2. Using let to create local variables
   1. A `let` expression is simply syntactic sugar for the underlying `lambda` application.

### 1.3.3 Procedures as General Methods

1. ???????????????????????????????????????(???????????????)????????????

### 1.3.4 Proceduces as Returned Values

1. first-class procedures
   1. In general, programming languages impose restrictions on the ways in which computational elements can be manipulated. Elements with the fewest restrictions are said to have *first-class* status.
   2. Lisp, unlike other common programming languages, awards procedures full first-class status.

# 2 Building Abstractions with Data

1. The general technique of isolating the parts of a program that deal with how data objects are represented from the parts of a program that deal with how data objects are used is a powerful design methodology called ***data abstraction***.

## 2.1 Introduction to Data Abstraction

1. Data abstraction is a methodology that enables us to isolate how a compound data object is used from the details of how it is constructed from more primitive data objects

### 2.1.1 Example:Arithmetic Operations for Rational Numbers

1. ????????????????????????selector???constructor??????getter???????????????

2. pairs:takes two arguments and returns a compound data object that contains the two arguments as parts

   1. ```scheme
      (define x (cons 1 2))
      
      (car x)
      1
      
      (cdr x)
      2
      ```

### 2.1.2 Abstraction Barriers

1. isolate different 'levels' of the system.
2. In effect, procedures at each level are the interfaces that define the abstraction barriers and connect the different levels
3. ??????????????????????????????????????????????????????????????????

### 2.1.3 What Is Meant by Data?

1.  In general, we can think of data as defined by some collection of selectors and constructors, together with specified conditions that these procedures must fulfill in order to be a valid representation
2. Message paasing:the ability to manipulate procedures as objects automatically provides the ability to represent compound data.

### 2.1.4 Extended Exercise:Interval Arithmetic

## 2.2 Hierarchical Data and the Closure Property

1. closure property:In general, an operation for combining data objects satisfies the closure property if the results of combining things with that operation can themselves be combined using the same operation

### 2.2.1 Representing Sequences

???????????????

### 2.2.2 Hierarchical Structures

???????????????????????????????????????????????????????????????

### 2.2.3 Sequences as Conventional Interfaces

1. modular
   1.  We can encourage modular design by providing a library of standard components together with a conventional interface for connecting the components in flexible ways.
2. ????????????????????????
3. nested mappings ???????????????

### 2.2.4 Example:A Picture Language

????????????????????????picture language



## 2.3 Symbolic Data

the ability to work with arbitrary symbols as data.

### 2.3.1 Quotation

```scheme
(define a 1)

(define b 2)

(list a b)
(1 2)

(list 'a 'b)
(a b)

(list 'a b)
(a 2)
```

# 2.4 Multiple Respresentations for Abstract Data

1. vertical barrier and horizontal barrier
   1. vertical barrier that gives us the ability to separately design and install alternative representations
   2. horizontal barrier isolate ``higher-level'' operations from ``lower-level'' representations.

### 2.4.1 Representations for Complex Numbers

????????????overload

### 2.4.2 Tagged data

????????????override

### 2.4.3 Data-Directed Programmingand Additivity

1. ?????????????????????not additivity
   1.  The person implementing the generic selector procedures must modify those procedures each time a new representation is installed, and the people interfacing the individual representations must modify their code to avoid name conflicts
2. data-directed programming
   1. nothing should change at all if a new representation is added to the system
   2. The key idea of data-directed programming is to handle generic operations in programs by dealing explicitly with operation-and-type tables
   3. ???table?????????????????????????????????????????????????????????????????????Message passing?????????????????????

## 2.5 Systems with Generic Operations

### 2.5.1 Generic Arithmetic Operations

???nested tag ???????????????isolation barrier

### 2.5.2 Combining Data of Different Types

cross-type operation

1. coercion???Often the different data types are not completely independent, and there may be ways by which objects of one type may be viewed as being of another type. This process is called coercion
2. ???????????????????????????
3. hierarchies of type
   1. ???????????? tower structure ?????????
   2. inadequacies????????????????????????????????????

# 3 Modularity,Objects,and State

1. Two "world views" strategies of the structure of systems
   1. Objects
      1.  viewing a large system as a collection of distinct objects whose behaviors may change over ti
   2. streams
      1. much as an electrical engineer views a signal-processing system.

## 3.1 Assignment and Local State

Java??????????????????????????????????????????????????????????????????????????? -> Objects???????????????

????????????Assignment???????????????????????????Objects???state change

### 3.1.1 Local State Variables

??????????????????????????????

### 3.1.2 The Benefits of introducing Assignment

By introducing assignment and the technique of one part of hiding state in local variables, we are able to structure systems in a more modular fashion than  if all stated to be manipulated explicitly, by passing additional parameters 

### 3.1.3 The Costs of Introducing Assignment

So long as we do not use assignments, two evaluations of the same procedure with the same arguments will produce the same result, so that procedures can be viewed as computing mathematical functions.

so called functional programming

