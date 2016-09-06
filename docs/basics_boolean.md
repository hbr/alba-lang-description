# Boolean and Boolean Logic

The module `boolean` is at the heart of the Albatross language. It is always
used either implicitely or explicitely. Its interface file is very short. We
can go through most of its declaration in this section.

## Boolean Type

At first it defines the type `BOOLEAN`

    class
        BOOLEAN
    end

The user knows of the type just that it is declared.

## Boolean Constants and Operators

After the type there are two constants declared in the interface file.

    false: BOOLEAN
    true:  BOOLEAN

The constants are just declared without any definition. Then the module
declares the basic boolean functions implication, conjunction and disjunction.

    (==>) (a,b:BOOLEAN): BOOLEAN
    (and) (a,b:BOOLEAN): BOOLEAN
    (or)  (a,b:BOOLEAN): BOOLEAN

These functions are just declared without any definition. Then boolean
negation and boolean equality are declared, but this time with a definition.

    (not) (a:BOOLEAN): BOOLEAN
        -> a ==> false

    (=) (a,b:BOOLEAN): BOOLEAN
        -> (a ==> b) and (b ==> a)

The user of the module `boolean` is aware of these definitions i.e. the
compiler is able to expand e.g. the expression `not (3 + 1 = 2)` to `3 + 1 = 2
==> false`.


## Precedence and Associativity

Implication has the lowest precedence, then follows `or` and then `and`. The
relational operators have higher precedence than the binary boolean operators
and the arithmetic operators have even higher precedence than the relational
operators. Boolean negation has higher precedence than the arithmetic
operators. This should coincide with the intuition one has usually about
precedences.

The implication operator `==>` is right associative. The implication chain `a
==> b ==> c` must be parsed as `a ==> (b ==> c)`. I.e. `a` implies the
implication `b ==> c`. Given `a` it is possible to conclude `b ==> c`. Give
`a` and `b` it is possible first to conclude `b ==> c` and then `c`.

The implication operator is the most important operator because the most
important reasoning of the [proof engine](proof_engine.md) is based on
implication.

The operators `and` and `or` have left associativity.


## Lazy Evaluation at Runtime

At runtime all binary boolean operators are evaluated lazily. I.e. if in the
expression `a and b` `a` evaluates to `false` then `b` is no longer
evaluated. The same applies to `a ==> b`. In both cases `b` can contain
undefined expressions as long as `a` evaluates to `false`. For disjunction the
opposite is the case. If in the expression `a or b` `a` evaluates to `true`
then `b` is no longer evaluated.


## Reasoning with Booleans

The boolean functions `==>`, `and` and `or` are given without definition. How
is it possible to reason about expressions containing these functions?

The logic for the implication operator is hardwired into the language and the
compiler. The proof engine can apply the modus ponens law i.e. given `a` and
`a ==> b` it concludes `b`. This is a kind of elimination rule for `==>`.

The boolean constant `true` and the operators `and` and `or` are defined in
the implementation file, but the definitions are not disclosed in the
interface file. Instead of a definition some theorems are listed in the
interface file which have been proved in the implementation file. For details
on the form of theorems see chapter [Theorems](theorems.md).

    all(a,b,c:BOOLEAN)
        ensure
            true

            a = a

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

            a or b
            ==> (a ==> c)
            ==> (b ==> c)
            ==> c                         -- or elimination

            a or not a                    -- excluded middle

            a or a ==> a                  -- idempotence of 'or'
        end

These theorems are sufficient to do any reasoning with boolean operators.

The negation laws allow us to do indirect proofs and conclude from falsehood
anything.

The elimination and introduction rules for `and` allow us to split up any
conjunction in its atomic parts and allow us to conclude conjunctions from its
atomic parts.

The introduction rule of `or` should be evident. The elimination rule for `or`
allows us to do proofs by case split. If we are able to proof the goal `c`
under the assumption of `a` and can prove it under the assumption of `b` and
we have the proposition `a or b` given then we can conclude `c` because one of
the assumptions is always true.

The law of the excluded middle asserts that boolean logic is a two valued
logic.


## Boolean Logic

The module `boolean` only contains the most basic laws about booleans. The
module `boolean_logic` uses these basic laws and derives some more laws abount
booleans. Therefore modules usually prefer to use `boolean_logic` instead of
the bare bone `boolean` to have more laws available to do reasoning.

<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
