# Inductively Defined Sets and Relations



## Inductive Definition

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
        = {(p): 0 in p, all(n) n in p ==> n + 2 in p}

    odd_numbers: ghost {NATURAL}
            -- The set of odd numbers.
        = {(p): 1 in p, all(n) n in p ==> n + 2 in p}


The two constants `even_numbers` and `odd_numbers` must be ghost constants
because no algorithm is given to decide if an element is in the set or not.

There are many sets which satisfy all the rules. E.g. the set of all natural
numbers satisfy all the rules of `even_numbers` and `odd_numbers`. The
important thing to remember is that an inductive definition defines
the **least** set which satisfies the rules.

An important consequence of this fact is that an element can be in an
inductively defined set only if it is in the set because of one of the rules
which define the set. E.g. `1` cannot be in the set of even numbers because it
is not `0` nor is there be any number `n` so that `n + 2 = 1`.




## Form of the Rules

Each rule of an inductive set definition is an implication chain of the form

    all(x,y...) c1 ==> c2 ==> ... ==> e in p

which states that a certain element `e` is in the set `p`. The variables
`x,y,...` and the premises `c1`, `c2`, ... are optional. The rule says that
`e` is in `p` either unconditionally or under certain conditions.


In the following we define some inductive sets using rules.

    A: ANY

    (+) (r:{A,A}): ghost {A,A}
            -- The transitive closure of 'r'.
        -> {(s): all(a,b)
                     r(a,b) ==> s(a,b)
                 ,
                 all(a,b,c)
                     s(a,b) ==> r(b,c) ==> s(a,c)
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
                     s(a,b) ==> r(b,c) ==> s(a,c)
            }

    path(a:A, f:A->A): ghost {A}
            -- The path starting with element 'a' and following the function 'f'.
        -> {(p): a in p
                 ,
                 all(a)
                     a in p ==> a in f.domain ==> f(a) in p
           }

The target always states that a certain element (maybe formed by an expression)
is in the inductive set (or relation). The conditions under which the element
is in the set might be that other elements are in the set as well or some
general conditions not referring to other elements of the set.

The conditions `ci` in a rule `all(x,y,...) c1 ==> c2 ==> ... ==> e in p`
might themselves be complete implication chains of the form `all(a,b,...) di1
==> di2 ==> ... ei in p` where the target always states that some other
element is in the set, possibly under the conditions `di1`, `di2`, ... which
must not contain the set `p`.

I.e. a rule in an inductive set has the general form

    all(x,y,...) c1 ==> c2 ==> ... ==> e in p

where each `ci` is either a general condition not referring to the defined set
or it has the form (or a degenerate variant)

    all(a,b,...) di1 ==> di2 ==> ... ==> ei in p

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
have predecessors then the set of accessible elements of this relation is empty.

If all elements of the carrier of a relation are accessible then there are no
infinitely descending chains in the relation i.e. the relation is wellfounded.


## Using the Rules

We know that `2` is an even number. Since it is not `0` there has to be some
`n` so that `n + 2 = 2` is valid. Obviously this `n` must be zero.

However the proof engine cannot prove `2 in even_numbers` fully automatically,
because it is unable to find this `n` so that `n + 2 = 2` is valid. In order
to prove `2 in even_numbers` we have to give the proof engine this number
explicitely.


    all  -- no variables needed
        ensure
            2 in even_numbers
        assert
            0 + 2 in even_numbers
        end

If the proof engine tries to prove `0 + 2 in even_numbers` it realizes that it
can use the second rule of the definition of `even_numbers` in backward
direction and try to prove `0 in even_numbers` which immediately succeeds by
the first rule. Having proved the assertion `0 + 2 in even_numbers` the proof
engine evaluates `0 + 2` to `2` and is therefore able to proof the goal `2 in
even_numbers`.

The proof engine is able to use the second rule of the definition of
`even_numbers` iteratively in backward direction because the rule is a
backward rule and it does not create subgoals which are more complicated than
the goal (details see chapter [Proof Engine](proof_engine.md)). Therefore the
following theorem is proved automatically by applying the second rule
iteratively in backward direction.

    all(n:NATURAL)
        require
            n in even_numbers
        ensure
            n + 2 + 2 + 2  in even_numbers
        end

In order to prove `n + 2 + 2 + 2  in even_numbers` it tries to prove `n + 2 +
2  in even_numbers`, then `n + 2 in even_numbers` and then `n in even_numbers`
which is identical to the precondition.


The second rule of the transitive closure `all(a,b,c) s(a,b) ==> r(b,c) ==>
s(a,c)` cannot be used in backward direction, because the target `s(a,c)` does
not contain all variables of the rule and the proof engine is unable to
_guess_ missing variables.

However it can be used in forward direction. As soon as there is some `s(a,b)`
in the context, the proof engine partially specializes the rule to `all(c)
r(b,c) ==> s(a,c)` which can be used in backward direction.

Assume that we know `r(a,b)`, `r(b,c)` and `r(c,d)`. This is sufficient to
conclude that `(a,d)` must be in the transitive closure of `r` i.e. that
`(+r)(a,d)` is valid. We can conclude this stepwise by using the following
proof.

    all(a,b,c,d:A)
        require
            r(a,b)
            r(b,c)
            r(c,d)
        ensure
            (+r)(a,d)
        assert
            (+r)(a,b)        -- 'r(a,b)' and the first rule
            (+r)(a,c)        -- partially specialized second rule and 'r(b,c)'
            (+r)(a,d)        -- partially specialized second rule and 'r(c,d)'
        end

Each of the intermediate steps `(+r)(a,b)` and `(+r)(b,c)` creates a partial
specialization of the second rule which can be used to prove the next step.



## Induction Proof

Inductively defined sets (and relations) can be used to do induction
proofs.

Above we made an inductive definition of the set of even numbers by stating
that `0` is an even number and for all even numbers `n` the number `n + 2` is
an even number as well.

However it is also possible to define a recursive function which _calculates_
if a number is even.

    is_even(n:NATURAL): BOOLEAN
            -- Is the number 'n' even?
        -> inspect
               n
           case 0 then
               true
           case 0.successor then
               false
           case n.successor.successor then
               n.is_even

Now we want to prove that for all numbers in the set `even_numbers` the
recursive function evaluates to `true`.

Let's analyze the situation. If `n in even_numbers` is valid then `n` has to
be in this set because of one of the rules which define the set
`even_numbers`.

The first rule states that `n` can be in the set if `n` is zero. In this case the
function `is_even` evaluates to `true`.

The second rule states that `n` can be in the set if there is another number
`m` which is in the set and `n` is `m + 2`. We can use the induction
hypothesis that `m.is_even` evaluates to `true`. Then `(m + 2).is_even`
evaluates to `true` as well by definition of `is_even`. Since `n` is `m + 2`
we have proved that `n.is_even` is valid.

The induction proof

    all(n:NATURAL)
        require
            n in even_numbers
        ensure
            n.is_even
        inspect
            n in even_numbers
        end

does exactly this reasoning automatically. The proof expression `inspect n in
even_numbers` instructs the compiler to analyze the different cases (rules)
under which `n` can be in this set.


Even if the proof engine of Albatross does the proof automatically we can
provide more details to make the steps more explicit. In the following proof
we trigger the different cases explicitely and mention the induction
hypothesis and the goals.



    all(n:NATURAL)
        require
            n in even_numbers
        ensure
            n.is_even
        inspect
            n in even_numbers
        case 0 in even_numbers
                -- 'n' is in 'even_numbers because it is zero.
            assert
                 0 in even_numbers         -- that is the case
                 0.is_even                 -- goal
        case all(m) m in even_numbers ==> m + 2 in even_numbers
                -- 'n' is in 'even_numbers' because there is some 'm' in
                -- 'even_numbers' and 'n' happens to be 'm + 2'.
            assert
                 m in even_numbers         -- because of the case
                 (m + 2) in even_numbers   -- because of the case
                 m.is_even                 -- induction hypothesis
                 (m + 2).is_even           -- goal
        end

This example demonstrates on how to write an induction proof on an inductively
defined set with explicit case analysis.

An analysis of a specific case is triggered by `case rule` where `rule` spells
out a specific rule of the inductive set. The programmer can freely choose the
names of the variables.

Within a case branch the compiler puts you in an environment with the
variables of the rule, all the facts which constitute the rule and for each
element which is already in the set because of some condition of the rule a
corresponding induction hypothesis.



## Induction Principle

Each inductively defined set has an induction principle which is a little bit
more complicated to understand than the induction principle associated with an
[inductive type](types_inductive.md).

The induction principle of an inductively defined set can be used to prove
that all elements of the set satisfy a certain property. In the following we
use the set `p` as a representant of the inductively defined set and the set
`q` as the set of all elements which satisfy the desired property.

If we want to prove that all elements of `p` satisfy `q` we basically want to
prove that `p` is a subset of `q`.

The set `p` is an inductively defined set i.e. the set `p` satisfies all rules
of the definition of `p`. We say that `p` _is closed_ under the rules `r1, r2,
...`.

The induction principle states that if the intersection `p * q` is closed
under the rules then `p` is a subset of `q`. Let's understand why this is the
case.

Firstly `p * q` is certainly a subset of `p` because every intersection of
sets is a subset of both sets.

Secondly by definition the set `p` is closed under the rules which define the
set, i.e. `p` is the least set which satisfies all rules. If `p * q` satisfies
the rules as well then `p * q` must be a superset of `p`.

If `p * q` is a subset and a superset of `p` then `p = p * q` is valid which
implies that `p` is a subset of `q` i.e. all elements of `p` satisfy the
predicate `q`.


Now let's look at the general form of a definition of an inductively defined
set.


    {(p): r1, r2, ... }

    ri: all(x,y,...) c1 ==> c2 ==> ... ==> e in p

    ci: all(a,b,...) d1 ==> d2 ==> ... ==> f in p

I.e. we can regard all rules `ri` and all conditions `cj`, `dk` as functions of
`p`.

If we want to prove that `p * q` satisfies the rules we have to prove for all
rules that

     all(x,y,...) c1(p*q) ==> c2(p*q) ==> e in p*q

is valid. Because of the form of the `ci` this condition is equivalent to

     all(x,y,...) c1(p) ==> c2(p) ==> ... ==> e in p
                  ==> c1(q) ==> c2(q) ==> ... ==> e in q

using the fact that `a and b ==> c` is equivalent to `a ==> b ==> c` and that
by definition `p` satisfies the rule i.e. `e in p` does not need to be proved
but can be assumed having `c1(p)`, `c2(p)`, ...

The conditions `c1(q)`, `c2(q)`, ... are induction hypotheses since they state
that we can assume that some other elements `f` already satisfy the predicate
`q` and we have to derive that `e` satisfies `q` from all premises in this
implication chain.

The clause `case all(x,y,...) c1 ==> c2 ==> ... ==> e in p` instructs the
compiler to analyze the case that `e` is in the inductively defined set `p`
because of this specific rule. The compiler the defines the variables
`x,y,...` of this specific rule and shifts the conditions `c1(p)`, `c2(p)`,
..., `e in p` and the induction hypotheses `c1(q)`, `c2(q)`, ... into to
context as assumptions and tries to prove the goal `e in q` from it.

Note that the goal predicate is of the form `{x: goal}` i.e. `e in q` is
by beta reduction equivalent to the goal where the induction variable `x` is
replaced by `e`.

Remember the example of the previous chapter where we wanted to prove `x in
even_numbers ==> x.is_even`. In this case the inductively defined set `p` is
`even_numbers` and the goal predicate `q` is `{x: x.is_even}`.

Recall the definition of `even_numbers`: `{(p): 0 in p, all(n) n in p ==> n +
2 in p}`.

The first case is trivial where the only assumption is `0 in even_numbers` and
the corresponding goal is `0 in {x: x.is_even}` i.e. `0.is_even`.

In the second case we have one variable `n` and one condition `n in p`. For
the condition we get a corresponding induction hypothesis of the form `n in
q`. I.e. for the second case we have to prove

     all(n)
         require
             n in even_numbers            -- c1(p)
             (n + 2) in even_numbers      -- e in p
             n.is_even                    -- c1(q)
         ensure
             (n + 2).is_even              -- e in q
         end


## More on Induction

In this section we use the following definitions for the set of even numbers
and the set of odd numbers.


    even_numbers: ghost {NATURAL}
        -> {(p): 0 in p, all(n) n in p ==> n + 2 in p}

    odd_numbers: ghost {NATURAL}
        -> {(p): 1 in p, all(n) n in p ==> n + 2 in p}

If we these definitions are inline with our understanding of even and odd
numbers we should be able to prove that both sets are disjoint i.e. we should
be able to prove `all(n,m:NATURAL) n in even_numbers ==> m in odd_numbers ==>
n /= m`.

In that example we have a number `n` which is in the set of even numbers and a
number `m` which is in the set of odd numbers. We can do induction on either
variable and in order to prove the claim that `n` and `m` are different we
have to do induction on both.

Let's use `n` for the outer induction proof and `m` for the inner induction
proof. Here is the skeleton if the outer induction proof.


    all(n,m:NATURAL)
        require
            n in even_numbers
            m in odd_numbers
        ensure
            n /= m
        inspect
            n in even_numbers
        case 0 in even_numbers
            ...
        case all(k) k in even_numbers ==> k + 2 in even_numbers
            ...
        end

In doing induction proofs with inductively defined sets it is very important
to know the generated goal predicate. For the outer proof the compiler generates
the generalized goal

    all(m) m in odd_numbers ==> n /= m

and the corresponding goal predicate `q`

    {n: all(m) m in odd_numbers ==> n /= m}

The second rule in the definition of `even_numbers` has the form `all(...) c1
==> e in p`. In this case we have `c1(p) = k in p` where `p` is
`even_numbers`.

In order to get the induction hypothesis we have to replace `p` by `q` and we
get

    k in {n: all(m) m in odd_numbers ==> n /= m}

which evaluates to

    all(m) m in odd_numbers ==> k /= m

We can fill the induction hypothesis into the skeleton of the proof.


    all(n,m:NATURAL)
        require
            n in even_numbers
            m in odd_numbers
        ensure
            n /= m
        inspect
            n in even_numbers
        case 0 in even_numbers
            ...
        case all(k) k in even_numbers ==> k + 2 in even_numbers
            assert
                all(m) m in odd_numbers ==> k /= m  -- outer induction
                                                    -- hypothesis
                m in odd_numbers   -- shifted into the context
                                   -- with a fresh variable 'm'

                -- goal: k + 2 /= m
                ...
        end

As already explained in the chapter [Inductive Types](types_inductive.md) the
compiler generates a generalized goal and a corresponding goal predicate but
in the cases of the induction prove it provides us with the corresponding
variables and the premises shifted into the context.

In our case the goal predicate is `{n: all(m) m in odd_numbers ==> n /= m}`
which generates for the second case the generalized goal `k + 2 in {n:
all(m) m in odd_numbers ==> n /= m}` or in evaluated form `all(m) m in
odd_numbers ==> k + 2 /= m`.

The compiler generates a fresh variable `m` which shadows the outer one,
shifts the premise `m in odd_numbers` into the context and tries to prove the
goal `k + 2 /= m`.

Now we can start with the inner induction in which we analyze the different
cases why `m` can be in `odd_numbers` and prove the inner goal `k + 2 = m`.

For the inner induction proof (induction on `m`) the compiler generates the
goal predicate `{m: k + 2 = m}` which does not need any generalization.

The second case of the inner induction proof is the more interesting one. In
the second case we analyze that `m` is equal to `o + 2` for some ther numbers
`o`. The goal in this case is `o + 2 in {m: k + 2 /= m}` which evaluates to
`k + 2 /= o + 2`. The induction hypothesis `o in {m: k + 2 = m}` which
evaluates to `k + 2 /= o` is useless in this case.

However we know that `o in odd_numbers` is valid and can specialized the
induction hypothesis of the outer induction proof to `k /= o` which is useful
because it implies `k + 2 /= o + 2`.

Having this we can complete the proof.

    all(n,m:NATURAL)
        require
            n in even_numbers
            m in odd_numbers
        ensure
            n /= m
        inspect
            n in even_numbers
        case 0 in even_numbers
            (inspect
                 m in odd_numbers
            )
        case all(k) k in even_numbers ==> k + 2 in even_numbers
            (assert
                 all(m) m in odd_numbers ==> k /= m  -- outer induction
                                                     -- hypothesis
                 m in odd_numbers   -- shifted into the context
                                    -- with a fresh variable 'm'

                 -- goal: k + 2 /= m
             inspect
                 m in odd_numbers
             case all(o) o in odd_numbers ==> o + 2 in odd_numbers
                 assert
                     k /= o          -- from outer induction hypothesis
                     k + 2 /= o + 2  -- goal
            )
        end

Note that the cases `0` and `1` are trivial because `0 /= 1` is valid and for
all numbers `x` the inequalities `0 /= x + 2` and `x + 2 /= 1` are valid which
can be verified by expanding the inequality function `/=` and by doing
computation of the equality function `=`.



<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
