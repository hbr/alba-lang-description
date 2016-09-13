# Inductively Defined Sets and Relations



## Inductive Definition of a Set

We can define sets by giving a start set and some rules on how to add elements
depending on the already available elements in the set.

E.g. we can define the set of even numbers by saying the `0` is an even number
and for every even number `n` the number `n + 2` is an even number as well.

Albatross has a special syntax to define sets inductively. E.g. the set of
even numbers as defined above in words can be defined as

    {(p:{NATURAL}):           -- type annotation optional
        0 in p,
        all(n) n in p ==> n + 2 in p}

The general syntax is

    {(p): rule_1, rule_2, ... }

Such an inductive definition defines the least set which satisfies all the
rules. The name of the set (here `p`) can be chosen arbitrarily.

As opposed to predicate expressions which have the form `{x: exp}` to define
the set of all elements which satisfy `exp`, an inductively defined set puts
the name in parentheses and the name does not represent an element, it
represents the whole set.

Usually an inductive definition of a set is used in a function or a constant
declaration.

    even_numbers: ghost {NATURAL}
            -- The set of even numbers.
        = {(p), 0 in p, all(n) n in p ==> n + 2 in p}

    odd_numbers: ghost {NATURAL}
            -- The set of odd numbers.
        = {(p), 1 in p, all(n) n in p ==> n + 2 in p}


The two constants `even_numbers` and `odd_numbers` must be ghost constants
because no algorithm is given to decide if an element is in the set or not.

There are many sets which satisfy all the rules. E.g. the set of all natural
numbers satisfy all the rules of `even_numbers` and `odd_numbers`. The
important thing to remember is that an inductive definition defines
the **least** set which satisfies the rules.

An important consequence of this fact is that an element can be in an
inductively defined set only if it is in the set because of one of the rules
which define the set. E.g. `1` cannot be in the set of even numbers because it
is not `0` nor can there be any number `n` so that `n + 2 = 1`.




## Form of the Rules

Each rule of an inductive set definition is an implication chain of the form

    all(x,y...) c1 ==> c2 ==> ... ==> e in p

which states that a certain element `e` is in the set `p`. The variables
`x,y,...` and the premises `c1`, `c2`, ... are optional. The rules say that
`e` is in `p` under certain conditions.


In the following we define some inductive sets using rules.

    A: ANY

    (+) (r:{A,A}): ghost {A,A}
            -- The transitive closure of 'r'.
        -> {(s): all(a,b)
                     r(a,b) ==> s(a,b)
                 ,
                 all(a,b,c)
                     s(a,b) ==> r(a,b) ==> s(a,b)
           }

    (*) (r:{A,A}): ghost {A,A}
            -- The reflexive transitive closure of 'r'.
        -> {(s): all(a)
                     a in r.carrier ==> s(a,a)
                 ,
                 all(a,b)
                     r(a,b) ==> s(a,b)
                 ,
                 all(a,b,c)
                     s(a,b) ==> r(a,b) ==> s(a,b)
            }

    path(a:A, f:A->A): ghost {A}
            -- The path starting with element 'a' and following the function 'f'.
        -> {(p): a in p
                 ,
                 all(a)
                     a in p ==> a in f.domain ==> f(a) in p
           }

The target always states that a certain element (maybe formed by an expression
is in the inductive set (or relation). The conditions under which the element
is in the set might be that other elements are in the set as well or some
general conditions not referring to other elements of the set.

The conditions `ci` in a rule `all(x,y,...) c1 ==> c2 ==> ... ==> e in p`
might be itself be complete implication chains of the form `all(a,b,...) di1
==> di2 ==> ... ei in p` where the target always states that some other
element is in the set, possible under the conditions `di1`, `di2`, ... which
must not contain the set `p`.

I.e. a rule in an inductive set has the general form

    all(x,y,...) c1 ==> c2 ==> ... ==> e in p

where each `ci` is either a general condition not referring to the defined set
or it has the form (or a degenerate variant)

    all(a,b,...) di1 ==> di2 ==> ... ei in p

    -- di1, di2, ... must not contain p

Using the general form we can define the set of all accessible elements of a
relation.

    accessibles(r:{A,A}): ghost {A}
            -- The set of accessible elements of the relation 'r'.
        -> {(p): all(y) (all(x) r(x,y) ==> x in p) ==> y in p}

This definition looks strange at first sight (a) because there are no start
elements for the set and (b) because it has some strange nesting of
quantifiers. However the definition of this set satisfies the general form of
an inductive definition of a set.

Let us try to understand this definition step by step.

The rule `all(y) ... ==> y in p` states that all `y` are in the set which
satisfy some condition.

The condition under which `y` is in `p` is that `all(x) r(x,y) ==> x in p` is
valid. I.e. `y in p` is valid only if all predecessors of `y` under the
relation `r` are in `p` as well.

Reading this carefully uncovers the secret. If an element `y` in the carrier
of `r` has no predecessors, then `y` must be in the set, because all its
predecessors (there are none) are in the set.

Now all elements can be added whose predecessors have no predecessors. This
process can be iterated until all accessible elements are exhausted.

If a relation has no elements which have no predecessors i.e. if all elements
have predecessors then the set of accessible elements of this set is empty.

If all elements of the carrier of a relation are accessible then there are no
infinitely descending chains in the relation i.e. the relation is wellfounded.


<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
