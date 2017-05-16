# BOOLEAN and ANY

##### Declaration

The interface file `core.ali` provides the following declarations.

    class BOOLEAN
    deferred class A:ANY

    class
        BOOLEAN
    inherit
        ANY
    end

    false: BOOLEAN
    true:  BOOLEAN
    (==>) (a,b:BOOLEAN): BOOLEAN
    (and) (a,b:BOOLEAN): BOOLEAN
    (or)  (a,b:BOOLEAN): BOOLEAN
    (not) (a:BOOLEAN):   BOOLEAN -> a ==> false
    (=)   (a,b:BOOLEAN): BOOLEAN -> (a ==> b) and (b ==> a)

    (=)  (a,b:A): BOOLEAN   deferred end
    (/=) (a,b:A): BOOLEAN   -> not (a = b)

    all(a,b:A)
        ensure
            a = a
            a /= a ==> false
            a = b or a /= b
        end

    all(a,b,c:BOOLEAN)
        ensure
            true
            -- negation
            (not a ==> false) ==> a       -- indirect proof
            false ==> a                   -- ex falso quodlibet
            -- conjunction
            a and b ==> a                 -- and elimination
            a and b ==> b
            a ==> b ==> a and b           -- and introduction
            -- disjunction
            a ==> a or b                  -- or introduction
            b ==> a or b
            a or b ==> (a ==> c) ==> (b ==> c) ==> c  -- or elimination
            a or not a                    -- excluded middle
            a or a ==> a                  -- idempotence of 'or'
        end

The module declares the classes `ANY` and `BOOLEAN` and their basic functions
equality `=`, inequality `/=`, implication `==>`, conjunction `and`,
disjunction `or` and negation `not`.

The class `ANY` is an abstract class, the keyword `deferred` is used to
express it and the class variable `A` is declared which represents for the
scope of the module any class which inherits `ANY` (more on abstract classes
and inheritance in the chapter [Abstract Classes](class_abstract.md)).

The class `BOOLEAN` inherits the class `ANY` i.e. it implements the equality
function require by its parent type and inherits the inequality function and
the properties of equality and inequality declared for the class `ANY`.

The class `BOOLEAN` has (constants and) functions which have no definition
(they are just declared) and two functions which are declared together with a
definition.

Boolean negation is defined as

    (not) (a:BOOLEAN): BOOLEAN -> a ==> false

This gives the proof engine the possibility to evaluate the
functions. Whenever it sees an expression of the form `not some_expression` it
can expand the definition and evaluate the expression to `some_expression ==>
false`.

Since `BOOLEAN` is a builtin type the compiler knows it and choses the most
effective instruction to evaluate a negated expression at runtime. The
definition given here is used to define the semantics of the function.


The implication operator `==>` is right associative. The implication chain `a
==> b ==> c` must be parsed as `a ==> (b ==> c)`. I.e. `a` implies the
implication `b ==> c`. Given `a` it is possible to conclude `b ==> c`. Give
`a` and `b` it is possible first to conclude `b ==> c` and then `c`.

The implication operator is the most important operator because the most
important reasoning of the [proof engine](proof_engine.md) is based on
implication.

The operators `and` and `or` have left associativity.


##### Lazy Evaluation at Runtime

At runtime all binary boolean operators are evaluated lazily. I.e. if in the
expression `a and b` `a` evaluates to `false` then `b` is no longer
evaluated. The same applies to `a ==> b`. In both cases `b` can contain
undefined expressions as long as `a` evaluates to `false`. For disjunction the
opposite is the case. If in the expression `a or b` `a` evaluates to `true`
then `b` is no longer evaluated.


##### Reasoning with Booleans

The boolean functions `==>`, `and` and `or` are given without definition. How
is it possible to reason about expressions containing these functions?

The logic for the implication operator is hardwired into the language and the
compiler. The proof engine can apply the modus ponens law i.e. given `a` and
`a ==> b` it concludes `b`. This is a kind of elimination rule for `==>`.

The boolean constant `true` and the operators `and` and `or` are defined in
the implementation file, but their definitions are not disclosed in the
interface file. Instead of a definition some theorems are listed in the
interface file which have been proved in the implementation file. For details
on the form of theorems see chapter [Theorems](theorems.md).

These theorems are sufficient to do any reasoning with boolean operators.

The negation laws allow us to do indirect proofs and conclude from falsehood
anything.

The elimination and introduction rules for `and` allow us to split up any
conjunction in its atomic parts and allow us to conclude conjunctions from its
atomic parts.

The introduction rule of `or` should be evident. The elimination rule for `or`
allows us to do proofs by case split. If we are able to proof the goal `c`
under the assumption of `a` and can prove it under the assumption of `b` and
we have already proved the proposition `a or b` then we can conclude `c`
because one of the assumptions is always true.

The law of the excluded middle asserts that boolean logic is a two valued
logic.


<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
