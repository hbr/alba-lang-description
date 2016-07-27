# A Quick Tour




## Packages and Modules

The basic compilation unit is the module. A module consists of an
implementation file (extension `.al`) and and interface file (extension
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

## Comments

    -- A single line comment

    {: A multiline comment

       It spans over several lines {: and can be nested :}
       until the end of comment marker is detected :}


## Expressions with Basic Types


### Boolean

The base library defines a lot of basic data types. E.g. the module
[`boolean`][boolean] defines the type `BOOLEAN` with all be usual operators.

    -- a, b, c: BOOLEAN

    a and b or c            -- 'and' binds stronger than 'or'

    a ==> b ==> c or a      -- implication is right associative and binds
                            -- weaker than 'and' and 'or'

    not (a and b)           -- negation has highest precedence

    a = b                   -- boolean equivalence

### Natural

The module [`natural`][natural] defines the type `NATURAL` which represent
arbitrary large natural numbers.

    -- a,b,c,n: NATURAL

    (a + b * c) ^ 2 ^ n    -- exponentiation is right associative and binds
                           -- stronger than '+' and '*'

    a + n <= b and b < c   -- highest binding arithmetic, then relational,
                           -- then boolean

    not (a <= b)           -- but: negation is strongest



### Tuples

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



### Predicate Type

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
    * ps               -- union of all sets in ps


### Function Type

The function type and its relation functions are defined in the modules
[`function`][function] and [`function_logic`][function_logic].

    -- Types
    NATURAL -> NATURAL

    NATURAL -> NATURAL -> NATURAL       -- a curried two argument function
                                        -- '->' is right associative

    (NATURAL,NATURAL) -> BOOLEAN        -- an uncurried two argument function

    -- Expressions
    n -> n + 2

    (n,m) -> n + m

    (n,m:NATURAL):BOOLEAN -> n <= m     -- type annotations are optional if
                                        -- the compiler can infer the types

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

This type has three constructors without arguments. We can define data types
with constructors with arguments.

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

The type variable `A` describes an arbitrary type i.e. a type which inherits
the type `ANY` (defined in the module [`any`][any] of the base library.

The expression

    tree(1,tree(2,leaf,leaf), tree(3,(tree(4,leaf,leaf),leaf),leaf))

defines the binary tree

                            1
                          /   \
                         2     3
                              /
                             4



## Function Declaration

Functions are declared by giving a signature (name, argument types and return
type) and an expression which defines the function.

    inorder (t:BINARY_TREE[A]): [A]
            -- The inorder sequence of the elements of the tree 't'.
        -> inspect
               t
           case leaf then
               []
           case tree(info,left,right) then
               inorder(left) + info ^ inorder(right)

    (-) (t:BINARY_TREE[A]: BINARY_TREE[A]
            -- The flipped tree 't'.
        -> inspect
           case leaf then
               leaf
           case tree(info,left,right) then
               tree(info, -right, -left)

Note that operators can be used as function names, but they must be put into
parentheses.

The compiler verifies that there is no illegal recursion i.e. that at least
one argument in a recursive call is (structurally) less than the original
argument.

It might be possible that a defining expression is not computable. In that
case the function has to be declared as a ghost function.

    is_transitive (r:{A,A}): ghost BOOLEAN
            -- Is the relation 'r' transitive?
        -> all(a,b,c) r(a,b) ==> r(b,c) ==> r(a,c)


Instead of giving an explicit expression to define a function, a function can
be defined by properties.

    origin (b:B, f:A->B): ghost A
            -- The value in the domain of 'f' which is mapped by 'f' to 'b'.
            -- 'f(b.origin(f)) = b
        require
            f.is_injective
            b in f.domain
        ensure
            f(Result) = b
        end

In this case the compiler verifies that there exists a value which satisfies
the properties and that the value is unique. Note that functions can have
preconditions. The above function would be illegal without the preconditions
because for arbitrary function is not invertible and it is only possible to
find the original value if the image lies in the range of the function.




[boolean]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/boolean.ali

[any]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/any.ali

[tuple]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/tuple.ali

[natural]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/natural.ali

[function]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/function.ali

[function_logic]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/function_logic.ali

[predicate]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/predicate.ali

[predicate_logic]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/predicate_logic.ali



<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
