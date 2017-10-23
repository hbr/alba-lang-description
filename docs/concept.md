
# Language Concepts

The Alba progamming language has the following main features:

- Statically typed
- Type inference
- Functional
- Imperative
- Object oriented
- Modular
- Information hiding
- Concurrent
- Completely verified

**Statically typed** means that each runtime object has a certain type like
`BOOLEAN`, `INTEGER`, `STRING`, ... which is known already at compile
time. This feature guarantees that it is impossible to write silly expressions
like adding a boolean value to a number. C like type casts are not
possible. If an expression has the type `STRING` you can apply only string
operations to it.


However it would be tedious to declare all variables with explicit types. The
Alba language has a **type inferer** which can infer the types from the
context. Only toplevel declarations like the declaration of a global function
needs an explicit type annotation. Sometimes the type of an expression cannot
be inferred unambiguously from the context. In these cases explicit type
annotations have to be added to disambiguate the situation.

Alba is a **functional language** in the sense that functions are first class
citizens which can be passed around arbitrarily. I.e. functions can be
argument and return type of functions. Furthermore Alba share with functional
languages like Haskell and OCaml the support for immutable types. An object is
immutable by default unless declared to have a mutable type.

Although having a strong functional component Alba supports **imperative
programming** as well. It is possible to declare buffers and arrays as mutable
structures with the possibility to assign array elements or appending
characters or strings to a string buffer.

In order to support powerful abstractions, Alba borrows form **object
oriented** languages the concept of abstract data types and inheritance. An
abstract type is declared by a deferred class, deferred functions and deferred
properties. Deferred classes and deferred functions have no definition and
deferred properties have no proofs i.e. they are treated as axioms. Other
classes can inherit an abstract class by defining the abstract functions and
proving the abstract properties.


Alba is completely **modular** in the sense that modules can be verified and
compiled by just using the exported features of the used modules. In order to
achieve this each module has an implementation file (`module_name.al`) and an
interface file (`module_name.ali`). The interface file declares all classes,
inheritance relations, features and assertions which are exported to other
modules.

There are several possible ways to **hide information**. In the interface file
of a module the progammer can decide not to export certain classes, functions
and assertions. Furthermore it is possible to declare a function with an
explicit definition term and later on hide the definition (either within the
implementation file or the interface file). The same applies to classes. A
class can be declared as an inductive class and the programmer can decide to
export the class but not its structure as an inductive class. In that case the
constructors become normal functions.


It is possible to define processes which execute concurrently (not yet
implemented in version 0.4). 


The compiler is **verifying** all modules completely. Specifically:

- All functions terminate and return the specified value. No endless loops are
  possible.

- The result of all functions is uniquely defined.

- All stated assertions can be proved.

- All redefinitions of functions are consistent with their original
  definitions.

- There are no deadlocks and no livelocks.

In order to achieve this the compiler has a theorem prover.
  












<!-- Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End: -->
