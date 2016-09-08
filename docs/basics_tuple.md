# Tuple


Tuples are inductive types (see chapter [Inductive Types](types_inductive.md)
for details). The module `tuple` defines the tuple type in the following
manner.

    -- file: tuple.ali
    A: ANY
    B: ANY

    class
        TUPLE[A,B]
    create
        tuple(first:A, second:B)
    end

Since tuples are used frequently there are some shorthands. `(A,B)` is a
shorthand for `TUPLE[A,B]` and `(a,b)` is a shorthand for `tuple(a,b)`.

`(A,B,C)` is parsed as `(A,(B,C))` and `(a,b,c)` is parsed as `(a,(b,c))`.



There are the functions `first` and `second` to extract the first and the
second element of a tuple.


    first(t:(A,B)): A
            -- The first element of the tuple 't'.
        -> inspect
               t
           case (a,_) then
               a

    second(t:(A,B)): A
            -- The second element of the tuple 't'.
        -> inspect
               t
           case (_,b) then
               b

Since the type `TUPLE` has only one constructor the inspect expressions for
pattern matching need only one case.

The compiler adds an equality function and an inheritance from `ANY` for all
inductive types automatically.

In order to access elements of n-ary tuples, the `first` and `second`
functions have to be applied iteratively. Examples:

    (a,b,c).first          =    a
    (a,b,c).second         =    (b,c)
    (a,b,c).second.first   =    b
    (a,b,c).second.second  =    b

    ((a,b),c).first        =    (a,b)
    ((a,b),c).first.first  =    a
    ((a,b),c).first.second =    b
    ((a,b),c).second       =    c


<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
