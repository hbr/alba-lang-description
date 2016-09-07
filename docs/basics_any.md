# Any

The module `any` defines the abstract type `ANY`. The role of the type `ANY`
is to define equality and inequality. The type `ANY` is inherited by all
useful types in Albatross.

The module `any` uses the module `boolean`.

    use
        boolean
    end


## Type

`ANY` is an abstract type and is declared in the following manner in the
module `any`


    deferred class
        ANY
    end

Abstract types are declared with the keyword `deferred`.

An abstract type cannot be used directly because there cannot be any object of
an abstract type. There can be only other types which inherit an abstract
type.

In order to represent types which inherit an abstract type, type variables can
be introduced.


    A: ANY

The type variable can be substituted by any type which inherits `ANY`.

## Functions

The module `any` defines two functions using the abstract type.

    (=)  (a,b:A): BOOLEAN
        deferred
        end

    (/=) (a,b:A): BOOLEAN
        -> not (a = b)

Note that the type `ANY` is not (and must not be) used in the declaration of
the functions. Only a type variable can be used which represents all possible
descendants of `ANY`.

Any type which inherits `ANY` has to provide a definition of `=` or redefine
it as deferred (in case of an abstract type). The inequality operator `/=` has
a definition in the module `any` which is inherited by any type which inherits
the type `ANY`.


## Deferred Assertions

The descendants of `ANY` are not only required to define equality. The
equality must satisfy the requirement of reflexivity which the module `any`
states as a deferred theorem/assertion.

    all(a:A)
        ensure
            a = a
        deferred
        end

The keyword `deferred` states that this theorem/assertion is not proved. It is
just assumed. All descendents of `ANY` either have to prove this theorem (with
their specific equality function) or redeclare the assertion as deferred (in
case of an abstract type).


## Proved Assertions

Two trivial facts are proved in the implementation file `any.al` and exported
in the interface file `any.ali`.

    all(a,b:A)
        ensure
            a /= a  ==>  false

            a = b  or  a /= b
        end

## The First Inheritance

The type `BOOLEAN` has an equality function which satisfies the requirement of
reflexivity (see chapter [Boolean and Boolean
Logic](basics_boolean.md)). Therefore it is entitled to inherit the type
`ANY`. The module `boolean` cannot state this inheritance relation because in
order to state it had to use `any` which would create a cyclic module
dependency.

Therefore the module `any` states this inheritance relation.

    class
        boolean.BOOLEAN
    inherit
        ANY
    end

The type `BOOLEAN` has to be used in qualified form. Otherwise the compiler
would assume that the module `any` tries to define a new type `BOOLEAN`. The
use of a qualified type is necessary if a module defines an inheritance
relation and the module is not the defining module of the inheriting type.

In the function definitions the type `BOOLEAN` can be used without
qualification because there is no ambiguity.





<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
