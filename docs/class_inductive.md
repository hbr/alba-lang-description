# Inductive Classes

## Declaration

In order to define an inductive class we need a class name, optional formal
generics and a set of constructors. The type `LIST` e.g. has all these
elements.

    A: ANY

    class
        LIST[A]
    create
        []
        (^) (head:A, tail:LIST[A])
    end

The class `LIST` is a generic class with one class variable. The class
variable represents any class which inherits `ANY`. Remember that `[a,b,c]`
can be used as a shorthand for `a ^ b ^ c ^ []`.

What does this definition mean?

There are two ways to construct a list. The expression `[]` creates an empty
list. The second constructor allows to construct a list from an element and an
already constructed list by using the expression `element ^ list`.

There is no other possibility generate a list. I.e. the universe of lists can
be constructed by using either `[]` or `^`.

The first constructor is a basic constructor because it does not need an
already constructed list to construct a new list. The second constructor is a
recursive constructor which does need an already constructed list to generate
a new list.

Every inductive class must have at least one basic constructor. All base
constructors must appear before recursive constructors in the class
declaration.

Inductive classes are not necessarily recursive. E.g. the class
[`TUPLE`](basics_tuple.md) has only one base constructor.

In the following we mainly use the list type to explain the concepts. However
the concepts apply to all other inductive classs.


## Pattern Matching

The fact that only the constructors can be used to construct an element of the
type implies that given some list `list` it must have been constructed by one
of the constructors. Pattern matching can be used to decompose a list with an
inspect expression.

    inspect
        list
    case [] then
        ...
    case head ^ tail then
        ...

The variable names in the case clauses can be freely chosen.

Inspect expressions can inspect arbitrarily deep, because `tail` is also list
which must have been constructed by one of the constructors. E.g. it is
possible to distinguish between an empty list, a one element list, a two
element list and some longer list.

    inspect
        list
    case [] then
       ...
    case [head] then      -- [head] is a shorthand for head ^ []
       ...
    case [h1,h2] then     -- shorthand for h1 ^ h2 ^ []
       ...
    case h1 ^ h2 ^ h3 ^ tail then
       ...

If we ommit in the above inspect expression one or more cases, then the inspect
expression is no longer exhaustive. E.g. if we ommit the last case, then the
inspect expression has the precondition `not (list as _ ^ _ ^ _ ^ _)`. The
compiler generates the precondition and tries to verify (i.e. proof) it. If
the compiler cannot verify the precondition, then the expression is invalid.


## Equality

The compiler generates for each inductive class an equality function. E.g. for
the list class the compiler generates an equality function like

    (=) (a,b: LIST[A]): BOOLEAN
        -> inspect
               a, b
           case [], [] then
               true
           case h1 ^ t1, h2 ^ t2 then
               h1 = h2  and  t1 = t2
           case _, _ then
               false

I.e. objects of an inductive class can only be equal if they have been
constructed by the same constructor using equal arguments. In all other cases
the objects cannot be equal.

There are some consequences of this definition.

**Injectivity**: If we know that the two lists `h1 ^ t1` and `h2 ^ t2` are equal,
then we can immediately conclude that the heads and the tails are equal. In
fact the [proof engine](proof_engine.md) concludes from `h1 ^ t1 = h2 ^ t2` the
two facts `h1 = h2` and `t1 = t2` automatically by using the equality
definition. This is done recursively i.e. from `a1 ^ b1 ^ t1 = a2 ^ b2 ^ t2`
the three facts `a1 = a2`, `b1 = b2` and `t1 = t2` can be concluded.

This mathematical fact is called _injectivity_ of constructors: Two objects of
an inductive class can only be equal if they have been constructed by the same
constructors with the same arguments.

**Inversion**:
On the other hand if two objects of an inductive class have been constructed
differently, they must be different. The two objects `h ^ t` and `[]` cannot
be equal, because the first has been constructed by `^` and the second one by
`[]`. Therefore the [proof engine](proof_engine.md) concludes from `h ^ t =
[]` immediately `false`.

Injectivity and inversion are used by the [proof engine](proof_engine.md) to do
forward reasoning i.e. to derive something from an already established
equality of two objects of an inductive class. However the above definition of
equality can be used to do backward reasoning as well.

**Backward Reasoning**:
In order to prove that the two expressions `h1 ^ t1` and `h2 ^ t2` are equal
it is sufficient to prove `h1 = h2` and `t1 = t2` individually. Whenever a
goal of the form `h1 ^ t1 = h2 ^ t2` has to be proved, the [proof
engine](proof_engine.md) does the appropriate split up of the goal into
simpler goals.




## Inheritance from `ANY`

The equality function generated by the compiler complies with the requirements
of the equality function defined in the module `any`. Therefore the compiler
generates for any inductive class an inheritance relation so that the class
inherits `ANY`.




## Induction Law

The compiler generates for each inductive class an induction law. E.g. for
lists it generates the following law.

    all(p:{LIST[A]})
        require
            [] in p
            all(head,list) list in p ==> head ^ list in p
        ensure
            all(list) list in p
        end

In words: For all properties if the empty list has the property and all lists
transfer the property for any new head element to the list prefixed with the
new head element, then all lists have this property.

The induction law is another way of saying that all possible lists can be
created by using pnly constructors. There is no other way to construct lists.

The induction principle for any inductive class can be generated mechanically
by the following pattern.

    all(p: {T})           -- 'T' is the inductive class, possibly generic
        require
            all(a1,a2,...) ra1 in p ==> ra2 in p ==> ... ==> c1(a1,a2,...) in p
            ...
            ...
            all(a1,a2,...) ra1 in p ==> ra2 in p ==> ... ==> cn(a1,a2,...) in p
        ensure
            all(t) t in p
        end

where there is one premise for each constructor `ci` and

    a1,a2,...         -- all arguments of the constructor

    ra1,ra1,...       -- recursive arguments of the constructor, i.e.
                      -- arguments of type T

    c1,c2,...         -- constructors of the type 'T'

If a constructor has no arguments, then the universal quantification is
dropped. If a constructor has no recursive arguments, then the corresponding
premise has no induction hypothesis. Otherwise there is one induction
hypothesis for each recursive argument.

Example `UNARY_NATURAL`:

    class
        UNARY_NATURAL
    create
        0
        successor(n:UNARY_NATURAL)
    end

    all(p:{UNARY_NATURAL})
        require
            0 in p
            all(n) n in p ==> n.successor in p
        ensure
            all(n) n in p
        end


Example `BINARY_TREE`:

    class
        BINARY_TREE[A]
    create
        leaf
        tree(info:A, left,right:BINARY_TREE[A])
    end


    all(p:{BINARY_TREE[A]})
        require
            leaf in p
            all(i,l,r) l in p ==> r in p ==> tree(i,l,r) in p
        ensure
            all(t) t in p
        end


Example `TUPLE`:

    A: ANY
    B: ANY

    class
        TUPLE[A,B]           -- (A,B) is a shorthand for TUPLE[A,B]
    create
        tuple(a:A, b:B)      -- (a,b) is a shorthand for tuple(a,b)
    end

    all(p: {(A,B)})
        require
            all(a,b) (a,b) in p
        ensure
            all(t) t in p
        end


## Induction Proofs

Assume that addition on unary natural numbers has the following definition.

    (+) (a,b:UNARY_NATURAL): UNARY_NATURAL
        -> inspect
               b
           case 0 then
               a
           case b.successor then     -- The inner 'b' shadows the outer 'b'
               (a + b).successor

> Note that the variable names in pattern matching expressions can be chosen
  arbitrarily. The second case of the inspect expression says that `b` has
  been constructed as the successor of some other natural which we call in
  this case `b` as well, even if it evidently is a different value (it is the
  predecessor of the outer `b`).




With this definition the equality `a + 0 = a` for all numbers `a` is immediate
by just evaluating the expression on the left hand side of the equation.

Furthermore the expression `a + b.successor` evaluates immediately to `(a +
b).successor` by the second case clause and therefore the equality `a +
b.successor = (a + b).successor` is valid for all numbers `a` and `b`.

However the equality `0 + a = a` for all numbers `a` cannot be evaluated since
the compiler cannot do pattern matching on the pure variable `a`. This
assertion needs a proof.

We know that the equality `0 + a = a` is true for all numbers `a` (if we have
defined addition consistent with our usual understanding of addition of
natural numbers). If we want to use the induction principle we have to extract
the property `p`. In this case the property which shall be satisfied by all
numbers is `{n: 0 + n = n}`.

If we check the premise `0 in {n: 0 + n = n}` corresponding to the constructor
`0` we get by beta reduction `0 + 0 = 0` which immediately evaluates to true
by using the definition of addition.

For the constructor `successor` we can assume `a in {n: 0 + n = n}` i.e. `0 +
a = a` and have to prove that this assumption implies `a.successor in {n: 0 +
n = n}` i.e. `0 + a.successor = a.successor`. The compiler evaluates this goal
to `(0 + a).successor = a.successor` and then by using the defintion of
equality to `0 + a = a` (injectivity) which is identical to the induction
hypothesis.

Both premises of the induction law are satisfied for the propery `{n: 0 + n =
n}`, therefore all numbers satisfy this property.

We can write this proof formally as follows (details see chapter
[Theorems](theorems.md) and [Proof Engine](proof_engine.md)).

    all(a:UNARY_NATURAL)
        ensure
            0 + a = a
        assert
            0 in {n:UNARY_NATURAL: 0 + n = n} -- depending on the context some
                                              -- type annotation might be
                                              -- necessary to avoid ambiguities

            all(a) a in {n: 0 + n = n} ==> a.successor in {n: 0 + n = n}

            a in {n: 0 + n = n}
        end

This pedantic proof has been included here to demonstrate the link to the
generated induction principle. However it is very tedious to write such
proofs. E.g. the expression `{n: 0 + n = n}` occurs very often. Furthermore
none of the premises of the induction law needs an explicit proof because all
is done by pure symbolic evaluation of terms as explained above.

All the steps above are just a mechanical application of the induction
principle. Fortunately the Albatross compiler can do all the tedious work for
you and you just write this proof as

    all(a:UNARY_NATURAL)
        ensure
            0 + a = a
        inspect
            a
        end

The proof expression `inspect a` instructs the compiler to look up the
induction principle for the type of the variable `a`, transform the goal `0 +
a = a` into a property (i.e. a predicate) and apply the induction law.




## More on Induction Proofs

Now let's try to proof commutativity of addition `a + b = b + a` for all
numbers `a` and `b`. We can do induction on `a` or on `b` and since the goal
is symmetric there should be no difference. So let's try induction on `b`.

Since we have already proved `a + 0 = a` and `0 + a = a` the case that `b`
has been constructed with `0` should be no problem.

The induction step tells us that we can assume the induction hypothesis `a + b
= b + a` and have to derive the goal `a + b.successor = b.successor + a` from
it.

We can evaluate the left hand side to `(a + b).successor`. However there is no
way to transform the right hand side `b.successor + a` into the form `(b +
a).successor` in order to evaluate equality and to use the induction
hypothesis. The successor function is applied to the wrong variable `b`. If it
were applied to `a` instead of `b` we could derive the desired equality.

Our intuition tells us that `a.successor + b` should be equivalent to `a +
b.successor`. An induction proof on this goal immediately succeeds which can
be verified by pure evaluation and using the alreay proved equalities with
`0`.

    all(a,b:UNARY_NATURAL)
        ensure
            a + b.successor = a.successor + b
        inspect
            b
        end

Now we can prove commutativity of addition by the following proof.

    all(a,b:UNARY_NATURAL)
        ensure
            a + b = b + a
        inspect
            b
        case b.successor
            -- induction hypothesis: a + b = b + a
            -- goal: a + b.successor = b.successor + a
            via [  a + b.successor        -- left hand side
                ,  (a + b).successor      -- def '+'
                ,  (b + a).successor      -- induction hypothesis
                ,  b + a.successor        -- def '+'
                ,  b.successor + a        -- previous theorem
                ]
        end

This proof uses some more proof elements because just instructing the compiler
to do an induction proof is not sufficient.

An induction proof on `b` is done so the two different ways to construct `b`
have to be investigated. The case `0` runs automatically as explained above
and therefore does not need to be mentioned in the proof.

The case to assume the goal for `b` and derive the goal for `b.successor`
needs some help from the programmer.

The clause `case b.successor` instructs the compiler to analyze the case that
`b` has been constructed by the successor constructor applied to some variable
which we can name arbitrarily. We have chosen the name `b` here since in the
proof of this case we do not need the outer variable `b` and therefore it is
no problem to shadow it.

The compiler generates the induction hypothesis and the goal mechanically from
the induction principle. The induction hypothesis is inserted into the context
so that the proof can use it.

The goal `a + b.successor = b.successor + a` is an equality which is a
transitive relation (see chapter
[Predicate](concept_builtin_predicate.md)). Therefore we can prove `a = z` if
we can prove the intermediate steps `a = b`, `b = c`, ... , `y = z`.

The proof expression `via [b, c, ..., y]` is a way to tell the compiler that
one wishes to exploit the transitivity of the equality in the goal `a = z` and
the compiler should prove the intermediate equalities step by step. It is not
necessary to mention `a` and `z` in the via expression, but they can be
included because `a = a` and `z = z` are valid by reflexivity.

The above proof instructs the compiler to transform `a + b.successor` step by
step into `b.successor + a` and all steps succeed as already explained.





## Generalizing the Induction Goal


Suppose that `X` is an inductive class and we want to prove the assertion

    all(x:X, y:Y, ...)
        require
            r1
            r2
            ...
        ensure
            goal
        ...
        end

Since `X` is an inductive class the compiler could extract the goal predicate
`{x: goal}` and could use the induction principle of the type `X` and try to
prove `all(x) x in {x: goal}` i.e. `all(x) goal`. However this usually won't
succeed because it is much more than we requested. We did not want that all
`x` satisfy the goal, but only all `x` which satisfy the preconditions should
satisfy the goal.

I.e. we want to prove the assertion `all(x) r1 ==> r2 ==> ... ==>
goal`.

However the optimum is still a little bit better. The compiler pushes also all
other variables `y,...` into the goal and tries to prove the modified goal

    all(x) all(y,...) r1 ==> r2 ==> ... ==> goal

i.e. it extracts the predicate

    {x: all(y,...) r1 ==> r2 ==> ... ==> goal}

and use it to apply the induction principle in order to prove `all(x) x in
{...}`.

This improved goal predicate results in a much stronger induction hypotheses
than the induction hypothesis from the naively extracted goal predicate.

This is one service that the compiler does with all induction proofs. It
extracts the most general goal predicate before it applies the induction
principle of the corresponding inductive class.

But there is another service of the compiler which makes it very convenient to
work with. For specific cases of an induction proof the compiler reproduces
the situtation of the outer assertion into the specific inner
assertion. I.e. it provides an environment with fresh variables `y,...` and
all assumptions `r1`, `r2`, ... valid in the inner context and tries to prove
the appropiately adapted goal within this environment.







<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
