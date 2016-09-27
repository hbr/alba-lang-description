# Proof Engine

## How the Proof Engine Works

The proof engine maintains a context. The contexts consists of all assumptions
(require clauses and all already proved assertions in an assertion block) and
all assertions which can be reached directly or indirectly by forward
reasoning.

Each goal is proved within a context.

1. First the prove engine tries to prove the goal directly.

2. If this is not possible it checks if the goal is an implication chain with
or without universal quantification. In this case the proof engine defines a
new context with all the variables of the universal quantification and shifts
all the premises into the context and applies forward reasoning to the
context. Then it tries to prove the target of the implication chain as a new
goal.

3. As a last option a goal can be proved by backward reasoning. Backward
reasoning means trying to find already proved assertions which are implication
chains where the goal can be unified with the target. The premises of the
implication chain are used as new subgoals which have to be proved in order to
prove the goal. Since more than one assertion suited for backward reasoning
can be available, backward reasoning has to proceed by exploring different
alternatives.


## Deduction Law

If we define the variables `a`, `b`, ..., make the assumptions `r1`, `r2`,
... and prove in this context `g` we have actually proved `all(a,b,...) r1 ==>
r2 ==> ... ==> g`.

Therefore a proof of the theorem

    all(a:A, b:B, ...)
        require
            r1
            r2
            ...
        ensure
            g
        ...
        end

is a proof of the assertion `all(a,b,...) r1 ==> r2 ==> ... ==> g`.

Thereore if we want to prove the assertion `all(a,b,...) r1 ==> r2 ==> ... ==> g`
we can define a context with the variables `a`, `b`, ... and the assumptions
`r1`, `r2`, ... and prove within this context the goal `g`.


## Available Assertions

The context of the proof engine consists of a set of assertions. Every
assertion can be viewed as a universally quantified implication chain of the
form

    all(a,b,...) p1 ==> p2 ==> ... ==> tgt

where the universal quantifier and the premises are optional. The most
degenerate form of an universally quantified assertion is just the target
without any premises and variables.

An assertion with universally quantified variables is called a _schematic_
assertion.

All schematic assertions can be specialized. A specialization is a
substitution of the variables (or part of the variables) by some expressions.

Available assertions are always normalized. Normalization means to things.

- The variables in the universal quantification appear in the same order as
  they appear in the implication chain.

- The target is not universally quantified. Universally quantified variables
  in the target are always _bubbled up_. By bubbling up we mean that an
  assertion of the form `all(a) p ==> all(b) tgt` is equivalent to `all(a,b) p
  ==> tgt` where name clashes can be avoid by renaming the variables if
  necessary.


## Direct Proofs

There are some goals which can be prove by the proof engine directly without
starting some complex reasoning machinery.

(1) Goal is in the Context

In the simplest case the goal is already in the context. Then there is nothing
to do. E.g. the trivial theorem

    all(a:BOOLEAN)
        require
            a
        ensure
            a
        end

is proved directly. The proof engine creates the variable `a` and shifts the
expression `a` into the context. Then it tries to prove the goal `a` and finds
out that `a` is already in the context and there is nothing more to do.




(2) Schematic Assertion Unifiable with the Goal

Next there is the possibility that there is some schematic assertion which
consists only of a target and the target can be unified with the goal.

E.g. we have the schematic assertion which states that equality is reflexive.

    A: ANY

    all(a:ANY)
        ensure
            a = a
        end

Now assume that the prove engine encounters the goal `n + m = n + m`. The
proof engine unifies the target `a = a` with the goal by substituting `a` with
`n + m` and it gets `n + m = n + m` into the context which proves the goal
directly.



(3) Equality Proof

Thirdly the proof engine can do equality reasoning if the goal is some
equality where the left hand side and the right hand side of the equality
differ by some subexpressions. E.g. if the goal is

     a + b + c = a + d + e

the proof engine finds out that the left hand side and the right hand side are
structurally identical and the different subexpression pairs are `b,d` and
`c,e`. If the proof engine finds the equalities `b = d` and `c = e` in the
context it can conclude the validity of the goal immediately by using the
Leibniz condition of equality (equals can be substituted by equals).

Note that the proof engine does not try to _prove_ `b = d` and `c = e` by some
complex machinery. It just does a look up in the context if it encounters
the needed equalities.



(4) Witness Search

If the goal is an existentially quantified assertion i.e. it has the form
`some(x,y,...) exp` then the proof engine searches in the context for
assertions which can be unified with `exp` i.e. it tries to find substitutions
for the variables `x`, `y`, ... so that the substitution applied to `exp`
results in an assertion already available in the context.

If the goal is

    some(x) a + x = b

and the following assertion is already in the context

    a + 2 * c = b

then the proof engine unifies `a + x = b` with `a + 2 * c = b` and finds `2 *
c` as a witness for `x`.



## Forward and Backward Rules

(1) Non Schematic Implication Chains

All proved implication chains which are not schematic i.e. which do not have
universally quantified variables can be used in forward and in backward
direction.

A non schematic implication chain has the form

    p1 ==> p2 ==> ... ==> tgt

If `p1` is a proved assertion the proof engine concludes `p2 ==> ... ==>
tgt`. This is an application of the _modus ponens_ law. As long as the
resulting assertion is an implication and the first premise is available in
the context, the modus ponens law can be applied iteratively.

On the other hand if the goal to be proved is identical with the target of an
implication chain, then the implication chain can be used in backward
direction. In order to prove the goal the proof engine generates the subgoals
`p1`, `p2`, ..., proves them and adds them to the context and then uses the
modus ponens law iteratively to prove the goal.


(2) Schematic Implication Chain as Backward Rule

A schematic implication chain has the form

    all(a,b,...)  p1 ==> p2 ==> ... ==> tgt

Such a schematic implication chain can be used in backward direction if the
target contains all the variables and target is not _catch all_.

An expression is _catch all_ either if it is a single variable (which must be
of boolean type in that case) or if it is an expression which contains only
variables. An expression contains only variables if it is of the form `x in p`
or `r(x,y,...)` and `x`, `y`, ... `p`, `r` are variables.

Typical assertions with _catch all_ target:

    all(a:BOOLEAN) (not a ==> false) ==> a

    all(n:NATURAL,p:{NATURAL}
        0 in p ==>
        (all(n) n in p ==> n.successor in p) ==>
        n in p

The first one shall not be used in backward direction because proofs by
contradiction shall be triggered explicitely. Otherwise the proof engine would
try a proof by contradiction on all goals.

The second one shall not be used in backward direction because proofs by
induction shall be triggered explicitely. Otherwise all goals of the form `x
in p` where `x` is of an inductive type would trigger an induction proof which
is an overkill.

If the proof engine tries to prove a goal it examines all schematic
implication chains. If it is possible to substitute the variables of the
target of the schematic implication chain by subexpressions of the goal
(i.e. if it is possible to unify the goal with the target), then it
instantiates the whole rule by this substitution and uses it as a backward
rule. I.e. it tries to prove the instantiated premises as subgoals and in case
of success uses forward reasoning to derive the goal from the subgoal by using
the instantiated rule and the modus ponens law.

Example of a backward rule:

    all(a,b:BOOLEAN) a ==> b ==> a and b

This is a schematic backward rule because the target `a and b` contains all
variables and it is not _catch all_.

If we have the goal `n = m + 2 and x in q` the rule can be instantiated to

    n = m + 2 ==> x in q ==> n = m + 2 and x in q

and the proof engine generates the subgoals `n = m + 2` and `x in q` which it
tries to prove.

The generated subgoals might trigger other backward rules therefore this
process of generating subgoals by backward rules can be repeated.

In order to avoid infinite backward reasoning the proof engine checks for any
applied schematic backward rule if some generated subgoal is more complicated
than the target. If this is the case the schematic backward rule is put onto a
_blacklist_. Rules on the blacklist cannot be applied more than once during
backward reasoning.

Note that more than one rule (schematic or non schematic) can be applicable
for backward reasoning. If this is the case then the proof engine creates
alternatives. Each applicable backward rule creates one alternative to prove
the goal. All alternatives are explored by the proof engine until one of the
alternatives can be used to generate a complete proof of the original goal. If
one of the alternatives leads to success the other alternatives are dropped.



(3) Schematic Implication Chain as Forward Rule

Backward reasoning is goal directed because it starts from the goal and
searches for applicable backward rules in order to generate subgoals. Forward
reasoning is not goal directed because it explores the available assumptions
and tries to conclude something from these. Forward reasoning is more
restricted in order to avoid useless reasoning.

A schematic implication chain can be used in forward direction only if it is
not a backward rule.

Simple implications which are not backward rules an simplify the expression
are always used in forward direction.

The following rules are used in forward direction.

    all(a,b:BOOLEAN) a and b ==> a

    all(a,b:BOOLEAN) a and b ==> b

These rules are not backward rules because the target do not contain all
variables. For all valid substitutions of `a` and `b` these rules create
conclusions which are _simpler_ than the premise.

As a consequence all conjunctions regardless how nested they are
will always be split up into its elementary parts. E.g. the conjunction `a and
(b and c and d) and (e and f)` will be splitted up by forward reasoning into
the elementary parts `a`, `b`, ... `f`.

The there are rules which can be partially specialized in forward
direction. E.g. the rule

    A: ANY

    all(a,b,c:ANY) a = b ==> b = c ==> a = c

is not a backward rule because the target does not contain all variables. The
first premise does not contain all variables either. But if there is an
expression of the form `k = n + x` in the context the rules can be partially
specialized to

    all(c) n + x = c ==> k = c

The new rule is a backward rule because the target contains all variables (the
only remaining variable is `c`). Whenever the proof engine encounters a goal
of the form `k = something` it can use this rule to generate the subgoal `n +
x = something`.

There is also a degenerate form of partial specialization which is useful. We
have the general rule

    all(a:BOOLEAN) false ==> a        -- Ex falso quodlibet

which says that everything follows from false. This is not a backward rule
because it is _catch all_. However it can be partially specialized in forward
direction. The premise `false` contains part of the variables (an empty part!)
and if `false` is in the context this rule will be partially specialized to

    all(a) a

by forward reasoning. As a consequence any goal immediately succeeds.

It should be clear why _ex falso quodlibet_ is not a valid backward rule. If
it were any goal would create an alternative with the subgoal `false` which is
an overkill.


Furthermore there is the general rule

    all(a:BOOLEAN) (not a ==> false) ==> a

which is not a backward rule because it is _catch all_. But this rule is
definitely simplifying in forward direction. As soon as `not a ==> false` is
in the context for any expression `a` the proof engine concludes `a` by
forward reaoning.


## Entering Goals

If the goal is a universally quantified expression and it cannot be proved
directly then the proof engine _enters_ the goal.

A universally quantified goal has the general form

    all(a,b,...) p1 ==> p2 ==> ... ==> tgt

where the premises might or might not be present.

If the goal cannot be proved directly the context is augmented with the
variables `a`, `b`, ... and the premises `p1`, `p2`, ... are shifted into the
context. Then the proof engine tries to prove `tgt` in this augemented
context. In case of success the proof engine uses the deduction rule to prove
the original universally quantified goal.



## Evaluation

The proof engine can evaluate expressions in forward and in backward
direction. It never evaluates

- implications: `a ==> b`

- conjunctions: `a and b`

- disjunctions: `a or b`

- existentially quantified expressions: `some(x,y,...) ...`

- universally quantified expressions: `all(x,y,...) ...`

All other expressions which are in the context or which are a goal are subject
to evaluation.

In order evaluate an expression the proof engine looks at the toplevel
function. If this function has a definition then the proof engine expands the
definition. This is done recursively until no further evaluation is possible.

Forward reasoning: If an expression in the context is fully evaluated then the
fully evaluated expression is added to the context. Note that the intermediate
steps are not added to the context, only the final result.

Backward reasoning: A fully evaluated goal is taken as one alternative to
prove the goal (there might be others).

The evaluation is done lazily e.g. arguments are evaluated only if needed.

`not exp` is evaluated to `exp ==> false` which is the final result of the
evaluation because implications are not evaluated.

`a ^ b ^ list1 + list2` is evaluated to `a ^ b ^ (list1 + list2)` because the
concatenation function of lists has a definition (see [List](basics_list.md))
and the pattern matching can be applied recursively.

If the definition term of a function starts with a branch expression (inspect
or if expression), then the proof engine checks if it can decide symbolically
which branch to enter. Only if it can decide which branch to enter, then it
continues with the evaluation. This decision might require evaluation of
arguments. If it cannot decide which branch to enter then it does not evaluate
the function.

If the definition term of a function start with an if expression `if cond then
... else ...` then the proof engine evaluates `cond` and searches for
`cond_eval` and `cond_eval ==> false` in the context. If it finds one of the
them then it continues with the evaluation of the corresponding branch. Note
that the proof engine only searches for the conditions, it does not try to
prove them.




## Simplification

An expression of the form `lhs = rhs` is a simplification if the right hand
side is simpler than the left hand side. E.g. the assertion `a + 0 = a` is a
simplification expression.

Whenever an expression has the subterm `a + 0` it can simplify this subterm to
`a`.

Simplifications are done in forward direction i.e. with all assertions in the
context and in backward direction i.e. with the goal.



## Complexity of a Term

In the previous sections we talked about simpler or more complex terms without
explaining what the complexity of a term is.

The complexity of a term is the number of its nodes when viewed as a
tree. Each function is a node, each argument is a node, each quantifier is a
node.

However the complexity of a term is its complexity with all functions
expanded. I.e. it is not the obvious complexity but the potential
complexity. This potential complexity needs to be measured because the proof
engine does function expansions.

The complexity measure is conservative because it is calculated with all
functions expanded even if the proof engine does not expand all functions as
explained in the section evaluation.




<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
