# Predicate and Predicate Logic


A predicate `p` over a type is a total function which maps elements `a` of the
type to the boolean value `p(a)`. The result `true` indicates that `a`
satisfies the predicate, `false` indicates that `a` does not satisfy the
predicate.

All elements which satisfy the predicate form a set. Therefore we can identify
each predicate with the set of all elements which satisfy the predicate.

## The Predicate Type

The class `PREDICATE` is generic and needs a type argument to give a predicate
type. E.g. the type `PREDICATE[NATURAL]` is a predicate over natural numbers.

The module `predicate` in its interface file `predicate.ali` starts with the
following declarations

    use
        any     -- module 'boolean' used implicitely
    end

    A: ANY

    class
        PREDICATE[A]
    inherit
        ghost ANY
    end

The type argument for the predicate has to be a descendant of `ANY`. Types
which do not inherit `ANY` cannot be used to form predicates. This is one of
the reasons why all useful classes must inherit `ANY` i.e. they must have a
definition of equality.

The declaration of `PREDICATE` contains an inherit clause which states that
`PREDICATE` itself inherits `ANY`. This is the reason why we can form
predicates over predicates. E.g. the type `PREDICATE[PREDICATE[NATURAL]]`
describes a collection of sets of natural numbers.

The inheritance relation is marked as a ghost inheritance. This is necessary
because the equality of predicates is not computable (see below).

Prediates are very frequently used in Albatross. Therefore there is a
shorthand. The type `{NATURAL}` is equivalent to `PREDICATE[NATURAL]` and the
type `{{NATURAL}}` is equivalent to `PREDICATE[PREDICATE[NATURAL]]`.




## Basic Predicate Functions

Now we come to some basic functions of `PREDICATE`.


    (in) (a:A, p:{A}): BOOLEAN
            -- Is 'a' an element of the set 'p'?

    (/in) (a:A, p:{A}): BOOLEAN
            -- Is 'a' not an element of the set 'p'?
        -> not (a in p)

    (<=) (p,q:{A}): ghost BOOLEAN
            -- Is `p` a subset of `q`?
        -> all(x) x in p ==> x in q

    (=)  (p,q:{A}): ghost BOOLEAN
            -- Are `p` and `q` equal sets?
        -> p <= q  and  q <= p

The operator `in` is given without any definition. The expression `x in p` is
equivalent to `p(x)`. The former emphasizes the fact that predicates represent
sets (i.e. the set of all elements which satisfy the predicate), the latter
emphasizes the fact that predicates are functions. It is usually better to use
`x in p` instead of `p(x)` because it is more intuitive.


The less-equal operator `<=` is used to define the subset function. A set (aka
predicate) `p` is a subset of the set `q` if all elements of `p` are contained
in the set `q`. This is exactly what the definition of the operator `<=`
says. Since the definition contains a quantified expression, the function has
to be marked as a ghost function (see chapter [Functions](functions.md) for
details).

The equality operator `=` says that two sets are equal if they are mutually
subsets. This function does not use quantified expressions in its definition
term but it uses the ghost function `<=`. Therefore it has to be marked as
ghost function as well.

The equality operator is definitely reflexive. Therefore `PREDICATE` is
entitled to inherit `ANY`. However since the equality function is a ghost
function the inheritance relation has to be marked as a ghost inheritance.


## Predicate Expressions

We have already seen expressions like `x in p`, `x /in p`, `p <= q`. However
all these expressions contain variables which are predicates.

There remains the question: _How can predicates be created?_. Answer: With
predicate expressions.

Since a predicate is a boolean valued function we can define a predicate by
defining a variable and use an expression which computes whether the variable
satisfies the expression. The general form of a predicate expression is

    {variable(s): boolean_expression}

The variables must be annotated with types only if the types cannot be
inferred. Usually the compiler can infer the type of the variable(s).

Since predicates are total functions the boolean expression must be well
defined for all variables. If an expression is not well defined for all
variables (i.e. it has preconditions) then appropriate guards have to be put
in front of the expression with preconditions.

Assume that `e2` has preconditions and `e1` evaluates to `true` only if the
preconditions are satisfied then the following two predicate expressions are
valid while a predicate expression without the guards would not be valid.

    {variable(s): e1 ==> e2}
    {variable(s): e1 and e2}

Since boolean expressions are lazily evaluated the expression `e2` is
evaluated only if `e1` evaluates to `true`.

The boolean expression within a predicate expression might or might not
contain the variable(s). Since there are only two boolean constants we can
define the empty and the universal set with a predicate expression.


    {x:NATURAL: false}        -- empty set of natural numbers

    {x:NATURAL: true}         -- universal set of natural numbers

The first expression is a function which maps each natural number to `false`
which says that no natural number can satisfy this predicate. The second
expression maps all numbers to true, therefore all numbers satisfy this
predicate.

The module `predicate` uses these expressions to define the empty and the
universal set in a generic way.


    empty: {A}
            -- The empty set.
        = {x: false}

    universal: {A}
            -- The universal set.
        = {x: true}

`empty` and `universal` are two generic constants of the module `predicate`
which return predicates to represent the empty and the universal set.

Note that the type annotation in the predicate expressions is no longer
necessary because the compiler infers the type of `x` from the fact that the
constants are declared with type `{A}`.


Predicate expressions can be used to express more interesting facts. It is easy
to define set intersection and set union using predicate expressions.

    (*) (p,q: {A}): {A}
            -- The intersection of the set 'p' with the set 'q'
        -> {x: x in p  and  x in q}

    (+) (p,q: {A}): {A}
            -- The union of the set 'p' with the set 'q'
        -> {x: x in p  or  x in q}

These functions are not ghost functions. They are perfectly computable.

The same applies to set complement and set difference.

    (-) (p:{A}): {A}
            -- The complement of the set 'p'
        -> x /in p

    (-)  (p,q:{A}): {A}
            -- The set 'p' without the elements of the set 'q'
        -> {x: x in p and x /in q}

The operator `-` can be used to define set complement and set difference. This
is possible because in Albatross only the function name together with its
signature has to be unique. Since the signatures of these two functions are
different the functions are different.

All the set algebra functions (union, intersection, complement, etc.) are
defined in the module `predicate_logic`. The module `predicate` just defines
the most basic things about predicates.




## Collection of Sets

We have already mentioned that it is possible to define collection of sets in
Albatross. This is possible because `PREDICATE` inherits `ANY`.

The type `{{A}}` represents a collections of sets of elements of type `A`. We
can also form a collection of collections of sets like `{{{A}}}`.

If we have a collection of sets i.e. a predicate of type `{{A}}` we can form
the union and the intersection of this collection of sets. The union of a
collection of sets is a set which contains all elements which are contained in
at least one set of the collection of sets. The intersection of a collection
of sets is a set which contains elements which are contained in all sets of
the collection.

It is easy to define the union and intersection of collection of sets in
Albatross.

    (+) (ps:{{A}}): ghost {A}
            -- The union of the collection of sets 'ps'.
        -> {x: some(p) p in ps  and  x in p}

    (*) (ps:{{A}}): ghost {A}
            -- The intersection of the collection of sets 'ps'.
        -> {x: all(p) p in ps  ==>  x in p}

Evidently these functions have to be marked as ghost functions since their
definition terms are not computable. Note that `+` and `*` can be used as
binary and as unary operators. Since the signatures of these functions are
different from all other functions using these operators there is no
ambiguity.


## Beta Reduction

For those familiar with lambda calculus it is not hard to see that predicate
expressions are lambda expressions. In lambda calculus the expression `{x:
true}` would be expressed as `lambda x. true`.

In lambda calculus expressions can be beta reduced. E.g.

    (lambda x. p) a    ~>  p[x:=a]

where `p[x:=a]` means that all occurrences of the variable `x` in the
expression `p` are substituted by `a`.

The Albatross compiler can use beta reduction to symbolically evaluate an
expression of the form `a in p` if `p` is given as a predicate expression.

E.g.

    a in {x: exp}

evaluates to

    exp[x:=a]            -- this is not Albatross syntax

The compiler can do this evaluation symbolically because beta reduction
requires only the substitution of a variable by an expression.



## Enumerated Sets

In mathematics it is possible to define a set by enumerating its
elements. E.g. `{a,b,c}` is the set which contains the elements `a`, `b` and
`c`.

In Albatross this is possible as well. However it is important to understand
what the expression `{a,b,c}` means in Albatross.

The module `predicate_logic` defines the function

    singleton(a:A): {A}
            -- The singleton set with the only element `a`.
        -> {x: x = a}

The Albatross parser has some macro mechanism which transforms expressions of
the form `{a,b,...}` into `a.singleton + b.singleton + ...`.

I.e. the expression `{a}` is a shorthand for `a.singleton` and the expression
`{a,b,c,...}` is a shorthand for `{a} + {b} + {c} + ...`.

Therefore the expression

    z in {a,b}

evaluates to

    z in {a} + {b}            -- arithmetic operator '+' binds stronger than
                              -- the relational operator 'in'

    z in a.singleton + b.singleton

    z in {x: x in a.singleton or x in b.singleton}

    z in a.singleton or z in b.singleton

    z = a  or  x = b


which is inline with our intuition about enumerated sets.


## Leibniz Equality

We have already seen that equality is defined in the module `any` and any type
inheriting `ANY` must define an equality operator which is reflexive in order
to be entitled to inherit `ANY`.

However this is not the whole story. In Albatross equality has much stronger
properties which are enforced by the compiler. In Albatross two expressions
can be equal only if they cannot be distinguished. I.e. if `a = b` is valid
then all predicates which are satisfied by `a` must be satisfied by `b` as
well. Otherwise the expressions would be distinguishable by some predicate.

This type of equality is generally called _Leibniz equality_. Having Leibniz
equality guarantees that equals can always be substituted by equals.

Since we need predicates to define Leibniz equality the corresponding law is
included in the module `predicate`. It reads

    all(a,b:A, p:{A})
        ensure
            a = b ==> a in p ==> b in p
        end

Remember that Leibniz equality is enforced by the compiler. Relying on Leibniz
equality the module `predicate` can prove in its implementation file that
equality is symmetric and transitive. The interface file of the module
`predicate` just states these facts as proved.

    all(a,b,c:A)
        ensure
            a = b  ==>  b = a               -- symmetric

            a = b  ==>  b = c  ==>  a = c   -- transitive
        end


## Relations with Predicates

Since tuples (see chapter [Tuple](basics_tuple.md)) inherit `ANY` it is
possible to form predicates over tuples.

Assume that there are the two type variables `A` and `B` which can be
substituted by any type inheriting `ANY`. Then the type `{A,B}` is a predicate
over tuples i.e. it represents a set of pairs which is a binary relation.

Note that such a definition has to use the module `tuple` to be valid and the
module `tuple` uses the module `predicate`. Therefore the module `predicate`
cannot define functions using relations without creating a circular module
dependency.

Therefore the base library has a module `relation` which uses `tuple` and
`predicate` and can therefore define functions using relations. E.g. the module
`relation` defines the following functions

    A: ANY
    B: ANY

    domain(r:{A,B}): ghost {A}
            -- The domain of the relation 'r'.
        -> {a: some(b) r(a,b)}

    range (r:{A,B}): ghost {B}
            -- The range of the relation 'r'.
        -> {b: some(a) r(a,b)}

    inverse(r:{A,B}): {B,A}
            -- The inverse of the relation 'r'.
        -> {b,a: r(a,b)}

It is not difficult to see that these definitions corresponds to the usual
definitions in mathematics. E.g. the domain is the set of all elements `a`
such that there is a corresponding element `b` and the pair `(a,b)` figures in
the relation.

As opposed to sets where it is usually more intuitive to write `a in p`
instead of `p(a)` with relations it is usually more intuitive to write
`r(a,b)` instead of `(a,b) in r`. But both versions are equally valid in
Albatross.






<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
