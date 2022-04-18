# Single Responsibility Principle

## definition

Every module, class or function in a computer program should have responsibility over a single part of that program's functionality, and should encapsulate that part



# Interface Segregation Principle

## definition

No code should be forced to depend on methods it does not use

# Dependence Inversion Principle

## definition

1. High level modules should not depend on low level modules; both should depend on abstractions.
2. Abstractions should not depend on details. Details(concrete implementations) should depend on abstractions

# Liskov Substitution Principle

## definition

If *S* is a subtype of *T*, then objects of tyoe *T* in a program may be replaced with objects of type *S* withourt altering any of the desirable properties of that program

子类不要修改父类的方法

# Open-closed Principle

## definition

software entities (classes, modules, functions…) should be open for extension, but cloesd for modification

# Law of Demeter

## definition

1. Each unit should have only limited knowledge about other units: only units "closely" related to the current unit
2. Each unit should only talk to its friends; don't talk to strangers
3. Only talk to your immediate friends