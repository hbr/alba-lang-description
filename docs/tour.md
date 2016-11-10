# A Quick Tour

This chapter gives a quick overview over the language features without
spelling out details.


## Packages and Modules

The basic compilation unit is the module. A module consists of an
implementation file (extension `.al`) and an interface file (extension
`.ali`).

Each module belongs to a package. A package or library is a set of modules.

Each module starts with a usage block listing the used modules. Example

    -- file: my_module_3.al
    use
        alba.base.predicate_logic
        alba.base.natural
        my_module_1
        my_module_2
        ...
    end

This module uses the modules [`predicate_logic`][predicate_logic] and
[`natural`][natural] of the Albatross base library and the modules
`my_module_1` and `my_module_2` of the current package.

After the usage block an arbitrary sequence of declarations follow. A
declaration is either a type or type variable declaration, a function
declaration or a theorem (see below).

The modules of a package must reside in one directory. The directory has to be
initialized as an Albatross directory by issuing the command

    alba init

on the command line.

After initialization the command

    alba compile

compiles all modules in the directory which need to be compiled. Dependencies
are handled by the compiler i.e. the compiler recompiles only modules which
have been changed since the last compilation or which use a module which has
been changed since the last compilation.

The command

    alba help

displays the available commands and its options.


## Comments

    -- A single line comment (the blank after -- is significant!)

    {: A multiline comment

       It spans over several lines {: and can be nested :}
       until the end of comment marker is detected :}




## Boolean Type

The base library defines a lot of basic data types. E.g. the module
[`boolean`][boolean] defines the type `BOOLEAN` with all the usual operators.

    -- a, b, c: BOOLEAN

    a and b or c            -- 'and' binds stronger than 'or'

    a ==> b ==> c or a      -- implication is right associative and binds
                            -- weaker than 'and' and 'or'

    not (a and b)           -- negation has highest precedence

    a = b                   -- boolean equivalence


## Any Type

The type `ANY` is defined in the module [`any`](any). It is an abstract type
whose only purpose is to define equality `=` and inequality `/=`. Abstract
types can be used to define type variables. The declarition

    A: ANY

defines the Type variable `A` which can be substituted by any type which
inherits `ANY` i.e. any type which has a definition of the equality operator
(and all meaningful types have an equality operator). The module `any` defines
equality as an abstract function.

    A: ANY

    (=) (a,b:A): BOOLEAN
        deferred
        end

i.e. a function which _defers_ the actual definition to its descendants. But
based on this abstract function it defines inequality

    (/=) (a,b:A): BOOLEAN
        -> not (a = b)

which is inherited by any type which inherits `ANY`.

Note that every specific definition of the equality operator in any specific
type must satisfy the leibniz property which says that equal objects must be
indistinguishable. The compiler verifies this property. Therefore it is always
possible to substituted equals for equals.


## Natural Type

The module [`natural`][natural] defines the type `NATURAL` which represents
arbitrary large natural numbers.

    -- a,b,c,n: NATURAL

    (a + b * c) ^ 2 ^ n    -- exponentiation is right associative and binds
                           -- stronger than '+' and '*'

    a + n <= b and b < c   -- highest binding arithmetic, then relational,
                           -- then boolean

    not (a <= b)           -- but: negation is strongest



## Tuple Type

Tuples are defined in the module [`tuple`][tuple]. The type
`(NATURAL,BOOLEAN,NATURAL)` is a 3-tuple and `(5,true,3)` is a corresponding
tuple expression. Tuples are right associative.

     (NATURAL,BOOLEAN,NATURAL)   -- is parsed as (NATURAL,(BOOLEAN,NATURAL))

     (5,true,3)                  -- is parsed as (5,(true,3))

The components of a tuple can be accessed via the functions `first` and
`second`.

     (5,true,3).first  = 5

     (5,true,3).second = (true,3)

     (5,true,3).second.first  = true

     (5,true,3).second.second = 3

However it is usually more convenient to use pattern matching to access the
elements of a tuple.

     inspect
         t             -- of type (A,B,C)
     case (a,b,c) then
         -- some expression using the components


## List Type

The list type is defined in the module [`list`][list] of the basic library.

    -- Types
    [NATURAL]            -- A list of natural numbers

    [NATURAL,BOOLEAN]    -- A list of tuples of natural numbers and booleans

    -- Expressions
    []                   -- The empty list

    1 ^ []               -- The list [1] i.e. the value 1 prepended in front
                         -- of the empty list.

    [1,2,3,4] = 1 ^ 2 ^ 3  ^ 4 ^ []    -- '^' is right associative

    - [1,2,3,4] = [4,3,2,1]  -- operator '-' reverses the list

    [1,2].head = 1

    [1,2].tail = [2]

    -- A:ANY, x:A,  a:[A]
    - a + [x] = - x ^ a      -- '^' binds stronger than '-'

    - a + - b = - (b + a)



## Predicate Type

A predicate is a boolean valued total function. It defines the set of all
elements which satisfy the predicate. The type and the basic functions with
predicates are defined in the modules [`predicate`][predicate] and
[`predicate_logic`][predicate_logic].

    -- Types
    {NATURAL}          -- A set of natural numbers

    {NATURAL,BOOLEAN}  -- A set of pairs of naturals and booleans i.e.
                       -- a relation between natural numbers and booleans

    {{NATURAL}}        -- A collection of sets of natural numbers

    -- Expressions
    {n: n < 5}         -- The set of natural numbers below 5

    {n:NATURAL: n < 5} -- Type annotations can be given explicitely

    3 in {n: n < 5}  = true

    5 in {n: n < 5}  = false

    {n: n < 5} * {n: 0 <= n}  -- Set intersection

    {n: n < 5} + {n: 6 < n}   -- Set union

    {2}                -- Shorthand for {n: n = 2}

    {2,3}              -- Shorthand for {2} + {3}

    -- ps: {{NATURAL}}   i.e. ps is a collection of sets
    + ps               -- union of all sets in ps

    * ps               -- the intersection of all sets in ps



## Function Type

The function type and its related functions are defined in the modules
[`function`][function] and [`function_logic`][function_logic].

    -- Types
    NATURAL -> NATURAL

    NATURAL -> NATURAL -> NATURAL       -- a curried two argument function
                                        -- '->' is right associative

    (NATURAL,NATURAL) -> BOOLEAN        -- an uncurried two argument function

    -- Expressions
    n -> n + 2                          -- an anonymous function mapping its
                                        -- argument 'n' to 'n + 2'

    (n,m) -> n + m

    (n,m:NATURAL):BOOLEAN -> n <= m     -- type annotations are optional and
                                        -- necessary only if the compiler cannot
                                        -- infer the types

    (n -> n + 2)(1) = 3

    (n -> n + 2).is_injective = true

    3.origin(n -> n + 2) = 1            -- if a function is injective then it
                                        -- is valid to apply the function
                                        -- 'origin' to get the preimage


## Type and Type Variable Declaration

A data type is declared with a class declaration.

    class
        COLOR
    create
        red
        green
        blue
    end

This type has three constructors without arguments. It is also possible define
data types with constructors with arguments.

    class
        OPTIONAL_NATURAL
    create
        nothing
        just (value:NATURAL)
    end

By using type variables it is possible to declare generic datatypes

    A: ANY

    class
        BINARY_TREE[A]
    create
        leaf
        tree(info:A, left,right: BINARY_TREE[A])
    end

The type variable `A` represents an arbitrary type i.e. a type which inherits
the type `ANY` (defined in the module [`any`][any] of the base library.

The expression

    tree(1,tree(2,leaf,leaf), tree(3,(tree(4,leaf,leaf),leaf),leaf))

defines the binary tree

                            1
                          /   \
                         2     3
                              /
                             4

and has the type `BINARY_TREE[NATURAL]`.



## Function Declaration

Functions are declared by giving a name, a signature (argument types and
return type) and an expression which defines the function.

    inorder (t:BINARY_TREE[A]): [A]
            -- The inorder sequence of the elements of the tree 't'.
        -> inspect
               t
           case leaf then
               []
           case tree(info,left,right) then
               inorder(left) + info ^ inorder(right)

    (-) (t:BINARY_TREE[A]): BINARY_TREE[A]
            -- The flipped tree 't'.
        -> inspect
               t
           case leaf then
               leaf
           case tree(info,left,right) then
               tree(info, -right, -left)


The compiler verifies that there is no illegal recursion i.e. that at least
one argument in a recursive call is (structurally) less than the original
argument.

It is possible that a defining expression of a function is not computable. In
that case the function has to be declared as a ghost function.

    is_transitive (r:{A,A}): ghost BOOLEAN
            -- Is the relation 'r' transitive?
        -> all(a,b,c) r(a,b) ==> r(b,c) ==> r(a,c)


Instead of giving an explicit expression to define a function, a function can
be defined by properties.

    origin (b:B, f:A->B): ghost A
            -- The value in the domain of 'f' which is mapped to 'b' by 'f'.
            -- I.e. 'f(b.origin(f)) = b'
        require
            f.is_injective
            b in f.range
        ensure
            f(Result) = b
        end

In this case the compiler verifies that there exists a value which satisfies
the property or properties and that the value is unique. Note that functions
can have preconditions. The above function would be illegal without the
preconditions because an arbitrary function is not invertible and it is only
possible to find the original value if the image lies in the range of the
function.

Functions defined by properties are always ghost functions because no
algorithm is given to compute the result.

The language supports object oriented notation and classical mathematical
notation of function applications. The following expressions are equivalent.

    f(x,y,...) = x.f(y,...)

    g(x) = x.g

    g(g(g(x)))  = x.g.g.g

The last example illustrates that object oriented notation can be useful to
avoid excessive parentheses.


## Theorem Declaration and Theorem Proving

A theorem expressed in Albatross has the following form

    all(a:A, b:B, ....)        -- Variables of the theorem
        require
            r1
            r2
            ...
        ensure
            exp                -- expression which is generally true assuming
                               -- r1; r2; ...
        -- optional proof
        end

Example:

    all(a,b:BOOLEAN)
        require
            not (a or b)
        ensure
            not a
        end

The compiler can prove this theorem automatically. It just requires the
expansion of boolean negation and two backward reasoning steps.

In the same manner `not (a or b) ==> not b` can be proved automatically.

The similar statement `not (a and b) ==> not a or not b` cannot be proved
automatically by the compiler. It needs some help with a user supplied proof.

    all(a,b:BOOLEAN)
        require
            not (a and b)
        ensure
            not a or not b
        via require
            not (not a or not b)  -- has the form 'not (x or y)', therefore
                                  -- the above theorem can be applied

The compiler is given the hint to prove the goal by assuming the opposite and
derive a contradiction. A proof of the form `via require x` instructs the
compiler to assume `x` and derive a contradiction and afterwards to use `x ==>
false` to prove the original goal.


In the previous chapter we have defined a function which flips a binary
tree. Evidently flipping a tree twice shall result in the original
tree. I.e. we want to prove `- - t = t` for any tree `t`. Since the flipping
function `-` is defined by recursion a proof by induction might prove the
desired goal.

    all(t:BINARY_TREE[A])
        ensure
            - - t = t
        inspect
            t
        end

An induction proof is triggered by `inspect x` where `x` has to be a variable
of an algebraic type. The compiler is instructed to perform a case split on
the constructors of the type.

For the constructor `leaf` the proof is trivial because `- leaf = leaf` by
definition.

For the constructor `tree` the proof is not that trivial, but succeeds
automatically. The compiler tries to prove `- - tree(i,l,r) = tree(i,l,r)` and
expands `- - tree(i,l,r)` to `- tree(i,-r,-l)` and then to `tree(i, - - l, - -
r)`. Then it compares this expression which the right hand side `tree(i,l,r)`
and derives the subgoals that `- - l = l` and `- - r = r` have to be
true. This succeeds immediately by looking at the induction hypotheses.

In the previous chapter we defined the function `inorder` which returns the
inorder sequence of a binary tree. It should be possible to prove that the
inorder sequence of the flipped tree is the reverse of the inorder sequence of
the original tree. I.e. we want to prove the assertion `(- t).inorder = -
t.inorder` for any binary tree `t`.

This proof is done by induction with some help for the constructor `tree`.

    all(t:BINARY_TREE[A])
        ensure
            (- t).inorder = - t.inorder
        inspect
            t
        case tree(i,l,r)
            via [(- tree(i,l,r)).inorder
                 (- r.inorder) + ([i] + - l.inorder)   -- def
                 (- r.inorder + [i]) + - l.inorder     -- assoc
                 (- i ^ r.inorder) + - l.inorder       -- - a + [x] = - x ^ a
                 (- (l.inorder + i ^ r.inorder))       -- - a + - b = - (b + a)
                 (- tree(i,l,r).inorder)]              -- def
        end

For the constructor `tree` we have to derive the equality

    (- tree(i,l,r)).inorder = - tree(i,l,r).inorder

We do this by using the fact that `=` is transitive and provide the
intermediate steps which transform the left hand side into the right hand side
of the desired equality. If we want to prove a relation of the form `r(a,z)`
with any transitive relation `r` we can do this by the proof expression `via [
b; c; ... ; y]` provided that the intermediate expressions `r(a,b)`, `r(b,c)`,
..., `r(y,z)` are all provable.







[boolean]: http://github.com/hbr/albatross/blob/master/library/alba.base/boolean.ali
[any]:     http://github.com/hbr/albatross/blob/master/library/alba.base/any.ali
[tuple]:   http://github.com/hbr/albatross/blob/master/library/alba.base/tuple.ali
[natural]: http://github.com/hbr/albatross/blob/master/library/alba.base/natural.ali
[list]:    http://github.com/hbr/albatross/blob/master/library/alba.base/list.ali
[function]:http://github.com/hbr/albatross/blob/master/library/alba.base/function.ali
[function_logic]:http://github.com/hbr/albatross/blob/master/library/alba.base/function_logic.ali
[predicate]: http://github.com/hbr/albatross/blob/master/library/alba.base/predicate.ali
[predicate_logic]: http://github.com/hbr/albatross/blob/master/library/alba.base/predicate_logic.ali





<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
