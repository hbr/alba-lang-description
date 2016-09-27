# Theorems

## General Form

A theorem has the general form

    all(a:A, b:B, ...)
         require
             pre_1
             pre_2
             ...
         ensure
             goal

         ...        -- optional proof

         end

If a theorem has no variables then the variable list can be ommitted with the
parentheses, however the keyword `all` is still necessary. The block of
preconditions is optional. A theorem must have a goal.

In an implementation file a theorem can have an optional proof but only one
goal, in an interface file proofs are not allowed (proofs are considered as
_implementations_) but there can be many goals (ensure clauses).

A theorem states that for all values of the variables `a,b,...` the conditions
`pre_1`, `pre_2`, ... imply the goal `goal`.


Each proof consists of an optional assertion block and an optional proof
expression.


## Assertion Block

An assertion block within a proof has the form

    assert
        assertion_1
        assertion_2
        ...


The assertions in the assertion block have to be provable by the [proof
engine](proof_engine.md) by using the assumptions `pre_1`, `pre_2`, ... and
all previous assertions of the assertion block.

An assertion in the assertion block can be a complete theorem, i.e. theorems
can be nested arbitrarily. If an assertion of an assertion block is a theorem
it can have its own proof.

In an inner theorem the variables and the require clauses are
optional. I.e. the most degenerate form of an inner theorem looks like

    ensure
        exp
    -- optional proof of 'exp'
    end

If an inner theorem has no variables then the variable list together with the
keyword `all` has to be dropped. An inner theorem without variables but with
preconditions has the form

    require
        r1
        r2
        ...
    ensure
        exp
    -- optinal proof of 'exp' assuming 'r1', 'r2', ...
    end

The purpose of an assertion block is to enrich the context with
assertions. Without an assertion block the goal must be provable by just using
the preconditions. With an assertion block all assertions in the block can be
used to prove the goal.


## Proof by Case Split

A case split is a proof expression of the following form.


    if a

        -- optional proof of the goal under the assumption 'a'

    orif b

        -- optional proof of the goal under the assumption 'b'


The case split expression instructs the compiler to do the following steps:

- Prove `a or b`

- Shift condition `a` into the context and prove the goal

- Shift condition `b` into the context and prove the goal

Since `a or b` is valid and the goal is provable assuming `a` and assuming
`b`, the goal is provable assuming `a or b` which is the case because of the
first step.


There is a variant of a case split expression where `b` is `not a`.


    if a

        -- optional proof of the goal under the assumption 'a'

    else

        -- optional proof of the goal under the assumption 'not a'

This proof expression looks like an ordinary if-expression. However there is
no keyword `then` and instead of an expression an optional proof
(i.e. optional assertion block and optional proof expression) can be used.




## Proof by Contradiction

Sometimes we want to proof a proposition `a` by assuming the opposite i.e.`not
a` and deriving a contradiction i.e. `false` from it. A proof by contradiction
uses the general theorem `all(a:BOOLEAN) (not a ==> false) ==> a`.

The Albatross programming language has syntax support for proofs by
contradiction. A proof by contradiction has the following form.

    via require
         exp

    -- optional proof of 'false'


The clause `via require exp` instructs the compiler to assume `exp` and derive
`false` from it. After having done this successfully the assertion `exp ==>
false` has been proved. With the help of this assertion the original goal must
be provable.

Usually `exp` is the negation of the original goal.



## Transitivity Proof

If the goal has the form `r(a,z)` where `r` is a transitive relation like `=`
or `<=` the transitivity law `all(x.y,z) r(x,y) ==> r(y,z) ==> r(x,z)` of the
relation can be used to prove `r(a,z)` by using some intermediate expressions
`b`, `c`, ... and prove `r(a,b)`, r(b,c)`, ..., `r(y,z)`.

A transitivity proof has the following form

    via [
        b
        c
        d
        ...
    ]

A transitivity proof instructs the compiler to do the following steps.

- Verify that the relation `r` in the goal `r(a,z)` is transitive

- Verify `r(a,b)`

- Verify `r(b,c)`

- ...

- Verify `r(y,z)`

If the relation `r` is not only transitive but also reflexive then the
endpoints `a` and `z` can be included optionally in the list because `r(a,a)`
and `r(z,z)` are valid as well. If the relation is not reflexive this is not
possible.


## Existential Proof

An existential proof has the form

    via some(a,b,...)
        exp

An existential proof instructs the compiler to do the following.

- Verify `some(a,b,...) exp`

- Add the variables `a,b,...` and shift `exp` into the context

- Prove the original goal under the assumption that there are variables
  `a,b,...` which satisfy `exp`



## Induction Proof

There are induction proofs for [inductive types](types_inductive.md) and
[inductively defined sets and relations](inductive_set.md).

The first one has the form

    inspect
        x        -- 'x' must be a variable of an inductive type
    case constructor_1
        ...      -- optional proof for the case that 'x' has been constructed
                 -- by the constructor expression 'constructor_1'
    ...  -- other cases

The cases are optional. If no cases are present the compiler tries to prove
the goal for all cases implicitely. Therefore explicit cases are usually only
mentioned if an explicit proof is necessary for the specific case.


The second one has the form

    inspect
        x in set
    case rule_1
        ...      -- optional proof for the case that 'x' is in the set because
                 -- of 'rule_1
    ...  -- other cases

`x` must be a variable and `set` must be an expression which evaluates to an
inductively defined set and `x` must be in this set.



## Chaining of and Nesting of Proof Expressions

Proof expressions can be chained and nested.


    all(a,b,...)
        require
            pre_1
            pre_2
        ensure
            goal
        assert
            ass_1       -- derive 'ass_1', 'ass_2', ... from the preconditions
            ...
        via some(x)     -- prove 'some(x) exp' and use it to prove the goal
            exp
        assert
            ...
        via require     -- assume 'cond' and derive 'false' from it and afterwards
            cond        -- prove the goal assuming 'cond ==> false'

        if cond_a       -- prove 'false' assuming 'cond_a'
            assert
                ...
            inspect n
        orif cond_b     -- prove 'false' assuming 'cond_b
            assert
                ...
        end




## Realistic Theorem

We demonstrate some of the proof expressions with a more realistic
example. Suppose we have the following definitions.

    A: ANY
    B: ANY

    (<=) (p,q:{A}): ghost BOOLEAN
            -- Is 'p' a subset of 'q'?
        -> all(x) x in p ==> x in q

    domain (r:{A,B}): ghost {A}
            -- The domain of the relation 'r'.
        -> {a: some(b) r(a,b)}

    range (r:{A,B}): ghost {B}
            -- The range of the relation 'r'.
        -> {b: some(a) r(a,b)}

    carrier (r:{A,A}): ghost {A}
            -- The carrier of the relation 'r'.
        -> r.domain + r.range


Obviously if a relation `r` is a subset (subrelation) of the relation `s` then
the carrier of `r` has to be a subset of the carrier of `s`.

The following proof expands first the definition of `r.carrier <= s.carrier`,
Then it makes a case split and uses an existential proof within each case with
some assertion block.

    all(r,s:{A,A})
        require
            r <= s
        ensure
            r.carrier <= s.carrier
        assert
            all(x)
                require
                    x in r.carrier
                ensure
                    x in s.carrier

                -- 'x' is in 'r.carrier', therefore it has to be in the domain
                -- or in the range of 'r'. I.e. we can prove by case split.
                if x in r.domain
                        -- if 'x' is in the domain of 'r' then there is some
                        -- 'y' with 'r(x,y)
                    via some(y)
                        r(x,y)
                    assert
                        s(x,y)    -- because of 'r is a subset of 's'
                        x in s.domain
                        x in s.carrier
                orif x in r.range
                        -- if 'x' is in the range of 'r' then there is some
                        -- 'y' with 'r(y,x)
                    via some(y)
                        r(y,x)
                    assert
                        s(y,x)    -- because of 'r is a subset of 's'
                        x in s.range
                        x in s.carrier
                end
        end





<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
