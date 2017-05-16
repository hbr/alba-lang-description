# List

A list is a central datastructure in all functional languages. Albatross has a
strong functional component and therefore has a list class defined in its base
library.

##### List Type

The module `core` of the base library defines the class in the following
manner.

    class
        LIST[A]
    create
        []
        (^) (head:A, tail:LIST[A])
    end

The operator `[]` is used to create an empty list. The expression `a ^ l`
adds the element `a` in front of the list `l`.

The operator `^` is right associative, i.e. `a ^ b ^ l` is parsed as `a ^ (b ^
l)`.

The operator `^` is an arithmetic operator which has higher precedence than
the addition and the multiplication operators. In order to add `3+4` at
the front of a list of naturals parentheses have to be used `(3+4) ^ natlist`.

Because lists are used frequently there are some shorthands. `[A]` is a
shorthand for `LIST[A]` and `[a,b,c]` is a shorthand for `a ^ b ^ c ^ []`.

The module `core` does not contain more declarations for lists. More
interesting functions are declared in the module `list`. All the following
subchapters explain functions declared in the module `list`. 

Browse the source code of the module [`list`](list) to see the complete user
interface.

##### List Functions

It is possible to extract the first element of a list and tail of a list
(i.e. the list without the first element) provided that the list is not
empty. The module `list` defines two partial functions to do this.

    head (a:[A]): G
            -- The first element of the list 'a'.
        require
            a as x ^ t
        ensure
            -> inspect
                   a
               case h ^ _ then
                   h
        end

    tail (a:[A]): [A]
            -- The list 'a' with the first element removed.
        require
            a as x ^ t
        ensure
            -> inspect
                   a
               case _ ^ t then
                   t
        end

Since `LIST` is a inductive type, recursive functions are easy to define. The
module `list` defines some basic list functions.


    (in) (x:G, a:[A]): BOOLEAN
            -- Is the element 'x' contained in the list 'a'?
        -> inspect
               a
           case [] then
               false
           case h ^ t then
               x = h or x in t

    elements (a:[A]): {A}
            -- The set of elements of the list 'a'.
        -> {x: x in a}

    (+) (a,b: [A]): [A]
            -- The concatenation of the lists 'a' and 'b'.
        -> inspect
               a
           case [] then
               b
           case h ^ t then
               h ^ (t + b)

    (-) (a:[A]): [A]
            -- The reversed list 'a'.
        -> inspect
               a
           case [] then
               []
           case h ^ t then
               - t + [h]



##### Lists and Higher Order Functions

Higher order functions are functions which receive functions as arguments or
return functions as results.

We can define a function which receives a list and a function from the list
element type to another type and which uses the function to map all elements
of the list to generate a new list.

    B: ANY

    [] (f:A->B, a:[A]): [B]
            -- The list 'a' where all elements are mapped by 'f'.
        require
            a.elements <= f.domain
        ensure
            -> inspect
                   a
               case [] then
                   []
               case h ^ t then
                   f(h) + f[t]
        end

It is very convenient to use the bracket operator `[]` to define this
function because then the expression `f[a]` maps the whole list `a`
elementwise by the function `f` and the notation is very intuitive.

Having this function we can map a list of naturals by the successor function.

    successor[[0,1,2]] = [1,2,3]

Note that `successor` is a defined function and not a function object as
required by the signature of `[]`. However the compiler converts the function
`successor` into the function object `n -> n.successor` automatically.

Another interesting higher order function is the function `folded` which folds
a folding function over a list.

A folding function for a list of type `[A]` is a binary function with the
signature `[B,A]->B`. I.e. it takes a value of type `B` and a list element of
type `A` and returns a value of type `B`. The folding needs a start value `s`
and then we get

    f.folded(s, [a,b,c])   = f( f( f(s,a), b), c)


The function `folded` is defined as follows.

    folded (f:(B,A)->B, s:B, l:[A]): B
            -- The function 'f' folded with start value 's' over the list 'l'.
        require
            f.is_total
        ensure
            -> inspect
                   l
               case [] then
                   s
               case h ^ t then
                   f.folded(f(s,h), t)
        end


If `f` is a binary left associative operator `op`, then we get

    (op).folded(s, [a,b,c]) =  s op a op b op c

i.e.

    (+).folded(a, [b,c,d])  =  a + b + c + d











[list]: http://github.com/hbr/albatross/blob/master/library/alba.base/list.ali



<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
