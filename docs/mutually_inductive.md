# Mutually Inductive Types

## Definition


### Example: n-ary Tree

In the chapter [Inductive Types](types_inductive.md) we saw how to define
inductive types with base constructors and recursive constructors. All the
definition of an inductive type has been contained in one class
declaration. With this language feature it has been possible to define tuples,
lists, unary naturals, binary trees, etc.

However it is not possible to define n-ary trees where `n` is arbitrary for
each node in the tree. The children of an n-ary tree form a list of trees, but
the most straightforward definition of an n-ary tree does not fit into the
usual pattern described in the chapter [Inductive Types](types_inductive.md).


    A:ANY

    class
        TREE[A]
    create
        tree (info:G, children: [TREE[A]])
           -- Note: [TREE[A]] is a shorthand for LIST[TREE[A]]
    end

The constructor `tree` seems to be recursive but it is not trivially recursive
because it has no argument of type `TREE[A]` but only an argument of type
`[TREE[A]]` which represents a list of trees and not a tree.

Furthermore the class `TREE` has no base constructor and therefore it seems to
be impossible to construct a first tree without having another tree.

But wait: We can construct an empty list of trees with `[]` and then construct
a tree with no children with `tree(i,[])`. By using this first tree we can
construct more complex trees step by step. E.g. the tree

                   a
           -----------------
           b       c       d
         -----
         e   f

can be constructed by the following steps.

    tree(e, [])
    tree(f, [])
    tree(b, [ tree(e,[])
            , tree(f,[])
            ])
    tree(c, [])
    tree(d, [])
    tree(a, [ tree(b, [tree(e,[]), tree(f,[])])
            , tree(c, [])
            , tree(d, [])
            ])


Arbitrary deep trees with any number of children can be constructed in this
manner.

Therefore the above definition of the class `TREE` is perfectly valid in Alba.

The types `TREE`[A]` and `LIST[TREE[A]]` can be considered as mutually
inductive types. They have the three constructors

    []: [TREE[A]]

    tree(i:A, f:[TREE[A]]): TREE[A]

    (^) (t:TREE[A], f:[TREE[A]]): [TREE[A]]

where the first one is a base constructor and the second and the third are
recursive constructors. The first and the third construct list of trees
(i.e. forests) and the second construct trees.



### General Pattern


In order to define the mutually inductive types `T1`,  `T2`,  `T3`, ... the
following class declarations have to appear in one type declaration block.

    class
        T1
    create
        c11(...)            -- No argument of type 'T1', 'T2', 'T3', ...
        c12(..., x:T2, y:T3, ...)
        ...
    end

    class
        T2
    create
        c21(..., x:T1, ...)
        ...
    end

    class
        T3
    create
        c31(..., x:T1, y:T2, z:T3, ...)
        ...
    end

    ...


At least one of the constructors `c11`, `c12`, ... , `c21`, ... , `c31`,
... has to be a basic constructor which has either no arguments or only
arguments not having any of the types `T1`, `T2`, `T3`, ...

All other constructors can have arbitrary argument types.

In the above tree example only the class `TREE[A]` has been declared
explicitly. It has not been necessary to declare the class `LIST[TREE[A]]`,
because the class `LIST[G]` already exists and the type `LIST[TREE[A]]` can be
obtained by substituting `G` by `TREE[A]`. A more explicit declaration would
look like

    class
        FOREST[A]
    create
        []
        (^) (t:TREE[A], f:FOREST[A])
    end

    class
        TREE[A]
    create
        tree(i:A, f:FOREST[A])
    end

which satisfies the general pattern of a declaration of mutually inductive
types with the constructors `[]`, `tree` and `^` where the first one is basic
and the others are recursive.

It is more convenient to use `LIST[TREE[A]]` instead of `FOREST[A]` because
all the functions and assertions available from the class `LIST` can be used.




## Recursion


Mutually inductive types can have mutally recursive functions. We demonstrate
a pair of mutually recursive functions by flipping a tree i.e. creating a
mirror image of a tree.

           -- tree 't'                          mirror image 't.flip'


                   a                                    a
           -----------------                    -----------------
           b       c       d                    d       c       b
         -----                                                -----
         e   f                                                f   e


The mirror image of a tree can be obtained by reversing the list of its
children and then flipping all the children recursively. We create a function
`flip` to flip a tree and a function `reverse` to reverse a list of trees and
flipping all its trees. Mutually recursive functions must be declared in the
same function declaration block.


    flip (t:TREE[A]): TREE[A]
            -- Reverse the list of the children of 't' and flip all children
        -> inspect
               t
           case tree(i,f) then
               tree(i, f.reverse)

    reverse(f:[TREE[G]]: [TREE[G]]
            -- Reverse the forest 'f' and flip all its trees
        -> inspect
               f
           case [] then
               []
           case t ^ f then
               f.reverse + [t.flip]

The functions are mutually recursive because

    flip calls:     reverse
    reverse calls:  reverse, flip

The function `flip` is indirectly recursive because it does not call itself
directly, only indirectly via `reverse`. The function `reverse` is directly
recursive because it calls itself directly and indirectly recursive because it
calls itself indirectly via `flip`.

However the recursion is wellfounded because each recursive call is made with
a structurally smaller argument than the original call.

- `t.flip` calls `reverse` on the children `f` of `t`. `f.reverse` calls
  `flip` on the first element of `f`. The first element of the children of `t`
  is structurally smaller than `t`.

- `f.reverse` calls `reverse` on the tail of `f` which is structurally smaller
  than `f`.

- `f.reverse` calls `flip` on the first element `t` of `f`. `t.flip` calls
  `reverse` on its children. The children of the first element of `f` is
  structurally smaller than `f`.

In this manner the compiler can check all possible recursive calls and verify
that in all possible recursive calls at least one argument is structurally
smaller than the original argument. The compiler rejects all recursions which
are not wellfounded.






## Induction

With simple inductive types the corresponding induction principle is
straightforward. In order to prove that all elements of a type satisfy a
property it is sufficient to prove that all elements resulting from basic
constructory satisfy the property and all elements resulting from recursive
constructors satisfy the property as well. For recursive constructors we could
use the induction hypothesis that all recursive arguments already satisfy the
property.

With mutual inductive types there cannot be a single property which all
elements of the mutually inductive types satisfy because the mutually
inductive types are different and there is no property which can be satisfied
by elements of different types. E.g. there is no property for trees and
forests.

Therefore the induction principle for mutual induction must contain one
property for each of the mutually inductive types. In our tree example we need
a property for trees and one for forests.

Having one property for each of the involved types makes it possible to
construct a mutual induction principle.


    all(  pf: {[TREE[A]]}      -- forest property (i.e. set of forests)
       ,  pt: {TREE[A]}        -- tree property (i.e. set of trees)
       ,  t: TREE[A]
       ,  f: [TREE[A]]
       )
        require

            [] in pf                                    -- constructor: []

            all(t,f) f in pf ==> t in pt ==> t^f  in pf -- constructor: ^

            all(i,f) f in pf ==> tree(i,f) in pt        -- constructor: tree

        ensure

            (all(f) f in pf) and (all(t) t in pt)

        end

The induction principle has one precondition for each constructor. The
constructor has to transfer the property from each recursive argument to the
newly constructed element. If each constructor transfers the corresponding
properties from its recursive arguments to the newly constructed element then
all elements satisfy their corresponding property.

The induction principle in words:

- If all empty forests satisfy `pf`

- and all trees `t` and forests `f` where `t` satisfies `pt` and `f` satisfy
  `pf` imply that `t^f` satisfies `pf`

- and all forests `f` satisfying `pf` imply that for all `i` `tree(i,f)`
  satisfies `pt`

then

- all forests satisfy `pf` and all trees satisfy `pt`.

The induction principle is valid because there is no other way to construct
forests or trees than by repeatedly applying the constructors `[]`,  `^` and
`tree`. A collection of sets `pf` and `pt` which contain all possible
constructed forests and trees respectively has to contain all possible forests
and trees.

The compiler constructs the induction principle of a collection of mutually
inductive types by generating one premise for each constructor.


## Proofs

In order to prove something about trees we have to prove something about trees
and forests in parallel.

In the following we prove that flipping a tree twice results in the original
tree i.e. that the flip operation is an involution. We want to prove

    t.flip.flip = t

for all trees `t`. It is easy to find a corresponding goal on forests. A forest
reversed twice should result in the same forest i.e.

    f.reverse.reverse = f

for all forests `f`.

We could try to prove both goals in parallel by using the mutual induction
principle. But in doing this you will realize that the following intermediate
lemma is needed.


    (a + b).reverse  =  b.reverse + a.reverse
        -- for all forests 'a' and 'b'


First we prove the intermediate lemma. Formally the intermediate lemma looks
like

    all(a,b: [TREE[A]])
        ensure
            (a + b).reverse  =  b.reverse + a.reverse

        inspect ...
            ...         -- proof see below
        end

Since the concatenation function `+` on lists is recursive in the first
argument a proof by induction on `a` might be a good start point.

In order to use the induction principle on forests and trees we need another
goal on trees. The above goal is just a goal on forests. But we cannot find a
goal for trees which uses the function `flip` and corresponds to the above
goal on forests.

Since forests are just list of trees we can try a proof with the usual
induction principle on lists.

We start our induction proof by

    inspect
        a

indicating that we want to do an induction on `a` which is of type
`LIST[TREE[A]]`.


We have to consider the different cases to construct the forest `a`. We can
construct it with the constructor `[]`.

    case []
        -- no induction hypothesis
        -- goal: ([] + b).reverse  =  b.reverse + [].reverse

In order to prove this goal the compiler does not need any help.

Next we can construct a forest by using the constructor `^` i.e. that `a` has
been constructed by prepending a tree to another forest.

    case t ^ a
        -- induction hypotheses
        --     (a + b).reverse = b.reverse + a.reverse
        --     true             trivially satisfied by 't'
        -- goal: (t^a + b).reverse  =  b.reverse + (t^a).reverse

> Note: The variable `a` shadows the corresponding outer variable and is
  different from it. This is not problem in induction proofs because outer
  versions of induction variables are never used within a case.

The goal in that case is an equality where we can use a _via_ proof to
transform the left hand side step by step into the right hand side.

        via [ (t^a + b).reverse
            , (a + b).reverse + [t.flip]           -- def '+', 'reverse'
            , b.reverse + a.reverse + [t.flip]     -- ind hypo
            , b.reverse + (a.reverse + [t.flip])   -- assoc of '+'
            , b.reverse + (t^a).reverse            -- def 'reverse'
            ]


Now we start the proof of the involution of `flip` and `reverse`, our main
theorem.


    all(t:TREE[A],  f:[TREE[A]])
        ensure

            (t.flip.flip = t)
            and
            (f.reverse.reverse = f)

        inspect

            t, f

          ...
        end

The statement `inspect t,f` instructs the compiler to do induction on both
variables. Since the corresponding types are mutually inductive the compiler
uses the appropriate induction principle. The goal has one component for trees
`t.flip.flip = t` and one component for forests `f.reverse.reverse = f`
therefore the compiler is able to extract the corresponding properties.

We have to analyze one case for each possible constructor.

    case []
        -- goal: [].reverse.reverse = []

This case is proved automatically by the compiler by expanding the definition
of `reverse`.

The next case of using the constructor `^` is not trivial.

    case t ^ f
        -- induction hypotheses:
        --    t.flip.flip = t
        --    f.reverse.reverse = f
        -- goal:
        --    (t ^ f).reverse.reverse  =  t ^ f

The goal is an equality and we transform the left hand side step by step into
the right hand side.

        via
            [ (t ^ f).reverse.reverse
            
            , (f.reverse + [t.flip]).reverse          -- def 'reverse'
            
            , [t.flip].reverse + f.reverse.reverse    -- interm lemma

            , [t.flip].reverse + f                    -- ind hypo 2

            , [t.flip.flip] + f                       -- def 'reverse'

            , [t] + f                                 -- ind hypo 1

              t ^ f                                   -- def '+'
            ]


The third case analyzes the constructor `tree`.

    case tree(i,f)
        -- induction hypothesis:  f.reverse.reverse = f
        -- goal:                  tree(i,f).flip.flip  =  tree(i,f)
        via
            [ tree(i,f).flip.flip
            
            , tree(i,f.reverse).flip       -- def 'flip'
            
            , tree(i,f.reverse.reverse)    -- def 'flip'
            
            , tree(i,f)                    -- ind hypo
            ]

The proof of the last case is spelled out in detail. But the compiler can do
all the steps without help. First it evaluates `tree(i,f).flip.flip =
tree(i,f)` to `tree(i,f.reverse.reverse) = tree(i,f)` and then it discharges
the goal by applying the induction hypothesis.




<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
