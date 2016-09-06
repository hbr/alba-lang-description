# Functions


## Functions with Definition Terms {#def_terms}

The simplest definition of a function consists of a function name, a signature
and a definition term. Assume the module has a type variable `A: ANY`. Then
a function computing the union of two sets can be defined as

    (+) (p,q:{A}): {A}
            -- The union of the two sets 'p' and 'q'.
        -> {x: x in p or x in q}

`(+)` is the function name, `(p,q:{A}): {A}` is the signature and `{x: x in p
or x in q}` is its definition term. Similarly we can define the empty set as

    empty: {A}
            -- The empty set.
        = {x: false}

A constant is a special case of a function which does not have
arguments. Instead of the arrow `->` the equal sign `=` is used because a
constant function does not map any arguments.

A function might have preconditions in case that it is not defined for
arbitrary values of its arguments. For functions defined with definition terms
the following general forms are possible:


    name: RT
            -- Some constant.
        = expression


    name (a:A, b:B, ...): RT
            -- Function without precondition.
        -> expression


    name(a:A, b:B, ...): RT
            -- Function with precondition(s).
        require
            precondition_1
            precondition_2
            ...
        ensure
            -> expression
        end





## Ghost Functions

It might be the case that a function is not computable. E.g. if the defining
expression of a function contains the universal or the existential quantifier
then the function is not computable. A function using non computable functions
in its definition term is not computable as well.

Functions which are not defined by a computable algorithm are ghost
functions. A ghost function is declared by putting the keyword `ghost` in
front of the result type.

A ghost functions is mathematically well defined but it lacks an algorithm to
compute its result given its arguments.

Example: The function which returns the union or the intersection of an
arbitrary collection of sets is not computable and therefore has to be
declared as a ghost function.

    A: ANY

    (+) (ps:{{A}}): ghost {A}
            -- The union of the collection of sets 'ps'.
        -> {x: some(p) p in ps and x in p}

    (*) (ps:{{A}}): ghost {A}
            -- The intersection of the collection of sets 'ps'.
        -> {x: all(p) p in ps ==> x in p}

    -- Note: {A} is the type of set of elements of type A; A can be any 
    --       type which inherits ANY.
    --       {{A}} is type of a collection of sets of elements of type A

In words:

- The union of a collection of sets is the set of all elements which are in at
  least one set of the collection of sets.

- The intersection of a collection of sets is the set of all elements which
  are in all sets of the collection of sets.



## Functions Defined by Properties

It is possible to define a function by properties which the result has to
satisfy as long as the result exists and is uniquely defined.

E.g. a function is injective or one-to-one if there is only one argument which
maps to the same result.

    A: ANY
    B: ANY

    is_injective(f:A->B): ghost
            -- Is the function 'f' a one-to-one mapping
        -> all(a1,a2) {a1,a2} <= f.domain ==> f(a1) = f(a2) ==> a1 = a2

In words: If two elements of the domain of `f` map to the same value, then the
two elements must be identical.

If a function is injective, then it is possible to map its result back to its
origin. I.e. we can define the function

    origin(b:B, f:A->B): ghost A
            -- The origin which is mapped to 'b' by the function 'f'.
        require
            f.is_injective
            b in f.range
        ensure
            Result in f.domain
            f(Result) = b
        end

The function `origin` is defined by properties. The result of the function
must be in the domain of `f` and `f` maps the result to `b`.

If the compiler sees this definition it verifies that the result exists an is
unique. In this particular case the verification is quite straightforward by
looking at the definition of `is_injective` and `range`. The function `range`
is defined as

    range(f:A->B): ghost {B}
        -> {b: some(a) a in f.domain and f(a) = b}

The second precondition of `origin` requires that `b` is in the range of `f`
which guarantees that there is some element in the domain which maps to
`b`. The first precondition requires that `f` is injective which guarantees
the uniqueness of the result.

Note that functions defined by properties are ghost functions because no
algorithm is given to compute the result.




## Recursive Functions

It is possible to define recursive functions in Albatross. Recursive functions
are guaranteed to terminate because the compiler checks that each recursive
call of a function uses at least one argument which is structurally smaller
than the argument of the original call.

The easiest recursive functions are functions using arguments of an inductive
type (see chapter [Inductive Data Types](type_inductive.md)) and pattern
matching.

Suppose we define positive numbers in the following manner

    class
        POSITIVE
    create
        1
        twice(p:POSITIVE)           -- two times the positive number 'p'
        twice_plus1(p.POSITIVE)     -- two times the positive number 'p' plus 1
    end

This is a binary definition of positive numbers.

     1           1
     2           1.twice
     3           1.twice_plus1
     4           1.twice.twice
     5           1.twice.twice_plus1
     6           1.twice_plus1.twice
     7           1.twice_plus1.twice_plus1
     8           1.twice.twice.twice
     ...
     32          1.twice.twice.twice.twice.twice

Note how object oriented notation helps to avoid excessive parentheses. In
mathematical notation the number 8 would be written as
`twice(twice(twice(1)))`.

For positive numbers we can define the successor function and addition
recursively.

    successor(p: POSITIVE): POSITIVE
        -> inspect
               p
           case 1 then
               1.twice
           case n.twice then
               n.twice_plus1
           case n.twice_plus1 then
               n.successor.twice

    (+) (a,b: POSITIVE): POSITIVE
        -> inspect
               b
           case 1 then
               a.successor
           case n.twice then
               a + n + n
           case n.twice_plus1 then
               (a + n + n).successor

The `inspect`-expression is used to do pattern matching. There are 3
possibilities to construct a number of type `POSITIVE` because the class
`POSITIVE` has 3 constructors.

In the successor function there is one recursive call in the third case of the
pattern matching. This case is entered if the positive number `p` has been
constructed as `n.twice_plus1` with some number `n`. The number `n` is
structurally less then `p` because `p` has been constructed by using `n` as an
argument. Therefore `n` can be used in the recursive call `n.successor`.

The addition function does structural recursion on its second
argument. Recursive calls happen in the second and in the third case. The
second argument used in the recursive calls is always structurally less than
the original argument `b`.

The albatross compiler can expand recursive functions if it can determine the
correct branch. It does expansion iteratively therefore it can do symbolic
computations. E.g. the following theorem (see chapter [Theorems](theorems.md))
can be proved automatically by the proof engine by using iterated function
expansion.

    all
           -- 3.twice = 4 + 2
        ensure
           1.twice.successor.twice = 1.twice.twice + 1.twice
        end

The right hand side of the equality is successively evaluated to

    1.twice.twice + 1.twice
    (1.twice.twice + 1) + 1
    1.twice.twice.successor.sucessor
    1.twice.twice_plus1.successor
    1.twice.successor.twice
    1.twice_plus1.twice



## More Pattern Matching

We can define division by 2 on positive numbers. However this is a partial
function because 1 cannot be divided by 2.

    half(p:POSITIVE): POSITIVE
        require
            not (p as 1)
        ensure
            -> inspect
                   p
               case n.twice then
                   n
               case n.twice_plus1 then
                   n
        end

The precondition says that the argument `p` must not have been constructed by
the constructor `1`.

Furthermore it is possible to do pattern matching on more than one
variable. We can define the function _less_equal_ on positive numbers by the
following recursive function.

    (<=) (a,b:POSITIVE): BOOLEAN
        -> inspect
               a, b
           case 1,_ then
               true
           case n.twice, m.twice then
               n <= m
           case n.twice_plus1, m.twice_plus1 then
               n <= m
           case n.twice, m.twice_plus1 then
               n <= m
           case n.twice_plus1, m.twice then
               n.successor <= m
           case _, _ then
               false

The anonymous variable `_` can be used to represent arbitrary values. By using
anonymous variables it is not neccessary to list all possible
combinations. Since `POSITIVE` has 3 constructors there are 9 possible
combinations. However the function just needs to distinguish between 6
cases. The first case implicitly includes all combinations of `1` with any
other constructor for the argument `b`. The last case is used as a catch-all
for all remaining cases.

In each recursive call it is guaranteed that at least one argument
structurally decreases.

The correctness (by correctness we mean that it corresponds to our
understanding of `<=` for numbers) of the definition is evident for most of
the cases. Just the fifth case might need some thinking. This case tries to
analyze if `2n + 1 <= 2m` is valid. This is equivalent to `2n + 2 <= 2m + 1`
which is equivalent to `2n' <= 2m + 1` where `n'` is the successor of
`n`. This is valid if `n' <= m` is valid.



<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
