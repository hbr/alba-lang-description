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

This module uses the modules `predicate_logic` and `natural` of the Albatross
base library and the modules `my_module_1` and `my_module_2` of the current
package.


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



### Predicate

A predicate is a boolean valued total function. It defines the set of all
elements which satisfy the predicate.


## Types and Type Variables

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
the type `ANY` (defined in the module `any` of the base library.

The expression

    tree(1,tree(2,leaf,leaf), tree(3,(tree(4,leaf,leaf),leaf),leaf))

defines the binary tree

                            1
                          /   \
                         2     3
                              /
                             4

## Functions








[boolean]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/boolean.ali

[any]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/any.ali

[tuple]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/tuple.ali

[natural]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/natural.ali

[function]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/function.ali

[predicate]: https://raw.githubusercontent.com/hbr/albatross/master/library/alba.base/predicate.ali



<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
