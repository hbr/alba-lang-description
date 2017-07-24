# Pseudoinductive Types

## Motivation

Pseudoinductive types are types which behave like inductive types, but don't
have an inductive implementation.

These types are useful in order to define numbers and strings which
have an efficient implementation but have all the comfort of inductive types
like recursive functions and induction proofs.

Implentations of numbers and strings by inductive types are notoriously
inefficient.

The most straightforward way to define natural numbers is by an inductive type
with one constructor for the number zero and one constructor to represent the
successor function. The number 100 would be represented in this implementation
by

    0.successor.successor. ..... .successor

i.e. a linked list of 101 linked objects.

This implementation can be improved by using a binary representation of
positive naturals with one constructor for the number 1, one constructor to
duplicate the number and one constructor to duplicate the number and add one
to it. The number 100 would be represented in this implementation by

    1.twice_plus1.twice.twice.twice_plus1.twice.twice

still needing 7 linked objects one for each bit.

A machine number having 32 or 64 bits in one word is a much more efficient
implementation for numbers and has very fast arithmetic operations executed by
the _arithmetic logic unit_ (ALU) of the processor.

However it is difficult to describe the semantics of machine numbers and their
arithmetic operations in a way which is easy to understand.

A similar problem applies to character strings. Strings could be represented
by lists of characters. However this representation requires a linked object
for each character in the string. Since we want to be able to store
arbitrarily long strings containing some megabyte of information efficiently
we want a more compact representation.

An array of characters or an array of segments of arrays of characters is a
very compact way to represent strings of arbitrary size. However in order to
describe the semantics of strings and its operations a list representation is
far more appropriate. Therefore we want a compact implementation of strings
which logically behave like lists of characters.


## Machine Number

In order to demonstrate the power of pseudoinductive types we define an
abstract type to represent machine numbers with modulo arithmetic.

    deferred class
        BOUNDED_NATURAL

    N: BOUNDED_NATURAL

    (=) (a,b:N): BOOLEAN     deferred end

    deferred class
         BOUNDED_NATURAL
    inherit
         ANY
    end


There is a number zero and a greatest number.

    0: N         deferred end

    greatest: N  deferred end

We have a successor function which wraps around at the greatest number
and a predecessor function which is the inverse of the successor function.
Furthermore we need an axiom stating that any number is different from its
successor.

    successor (n:N): N deferred end

    predecessor (n:N): N   deferred end

    all (n:N)
        ensure
            n = greatest ==> n.successor = 0
            n.successor.predecessor = n
            n /= n.successor
        deferred
        end

It is convenient to define some other numbers.

    1: N  =  0.successor
    2: N  =  1.successor


## Induction Law

In order to do induction we need an induction law.

    all(n:N, p:{N})
        require
            0 in p
            all(n) n /= greatest ==> n in p ==> n.successor in p
        ensure
            n in p
        deferred
        end

The premises of this law state that there are two ways to construct an
abstract machine number.

- By the constant `0`

- By applying the successor function to an arbitrary number `n`
  different from the greatest number.

Any property which satisfies both premises is satisfied by all numbers. The
induction law guarantees that by using the constructors `0` and `successor`
all numbers can be constructed.

The second premise of the induction law has been chosen deliberately with the
condition `n /= greatest` to make both cases non overlapping.


**General pattern**:

An induction law for an arbitrary type `T` has the form


    all(p:{T}, x:T) pp1 ==> pp2 ==> .... ==> x in p


where the premises `pp1`, `pp2`, ... have the form

    all(a1,...) cond ==> ra1 in p ==> ra2 in p ==> ... ==> c(a1,...) in p

    -- a1,... :   arguments of the constructor 'c'
    -- ra1,...:   arguments of the constructor 'c' with type 'T'
    -- cond:      any condition on the arguments not involving 'p'


The extra condition `cond` is optional.

The compiler recognizes an induction law by syntactic analysis and stores the
induction law in the type.

Since the abstract type of bounded naturals has an induction law, we can use
it to prove that for every number `a` there exists another number `b` such
that `b.successor = a` is valid.

    all(n:N)
        ensure
            some(k) k.successor = n
        inspect
            n
        case 0
            assert
                greatest.successor = 0
        case m.successor
            assert
                m.successor = m.successor
        end

The induction law guarantees that all numbers can be constructed by repeated
application of two constructors `0` and `successor`. It does not provide any
facility for the compiler on how to decide which constructor has been used to
construct a number and on how to extract the arguments of the constructor. In
order to do that we need recognizers and projectors.


## Recognizer

Remember that a premise in the induction law of a pseudoinductive type has the
form

    all(a1,...) cond ==> ra1 in p ==> ... ==> c(a1,...) in p

with the optional additional condition `cond`. Such a premise is used by the
compiler to identify the constructor `c`.

A recognizer is a computable expression which given any value `x` of a
pseudoinductive type `T` allows to decide if the value has been constructed by
`c`.

If the constructor `c` is a constant, then the recognizer is the trivial
condition

    x = c

provided that equality for the type is not a ghost function.


If the constructor has arguments and there exist arguments `a1`, `a2`,
... such that `x = c(a1,a2,...)`, then `x` might have been constructed by the
function `c`. The compiler uses the optional additional condition `cond` to
extract the noncomputable recognizer condition

    some(a1,a2,...) cond and x = c(a1,a2,...)

A recognizer is a computable expression which is equivalent to this
condition. I.e. an assertion of the form

    all(x:T)  exp = (some(a1,a2,...) cond and x = c(a1,a2,...))
    -- exp: computable expression containing 'x'

indicates to the compiler that `exp` is a candidate for a recognizer condition
for the constructor `c` provided that it is not a ghost expression.

The different cases have to be disjoint. In order to guarantee disjointness
the compiler expects to find the assertions

    all(a11,a12...,a21,a22,...) rec_i ==> rec_j ==> false

where `rec_i` and `rec_j` are recognizer conditions for the different
constructors `c_i` and `c_j`.

Let us find the recognizers for our model of machine numbers. Since one of the
constructors is the constant zero the trivial recognizer condition is

    n = 0

For the constructor `successor` the compiler extracts from the induction
principle the noncomputable condition

    some(k) k /= greatest  and  n = k.successor

Since we have already proved that for any number `n` there exists a number `k`
such that `k.successor = n` we just have to exclude that `k` is the greatest
number. This can be guaranteed if `n` is different from zero.

    all(n:N)
        ensure
             (n /= 0) = (some(k) k /= greatest and n = k.successor)
        assert
             ...  -- proof needed
        end

In order to prove this assertion the forward and the backward direction of the
boolean equivalence have to be proved. The proof is a standard excercise and
left out here.

Furthermore we have to make sure that both recognizer conditions are
disjoint.

    all(n:N)
        require
            n = 0
            n /= 0
        ensure
            false
        end

This condition is trivially valid.

That completes the implicit declaration of the two recognizers `n = 0` and `n
/= 0`. Both conditions are obviously computable. The compiler is allowed to
compile an as expression of the form `n as 0` into `n = 0` and an as
expression of the form `n as successor(_)` into `n /= 0`.


## Projector

In order to convert inspect expressions of the form

    inspect
        x
    case c1(a11,...) then
        exp_1(a11,...)
    case c2(a21,...) then
        exp_2(a21,...)
    case c3(a31,...) then
        exp_3(a31,...)

into some computable expression it is not sufficient that the compiler has
computable expressions to decide the case, it must have computable
expressions to extract the arguments of the constructors as well.

An expression which extracts an argument of a constructor is called a
projector. Obviously a projector `proj_ij` which extracts from the i-th
constructor the j-th argument must satisfy the condition

    all(a_i1,...,a_ij,...)
         ensure
             proj_ij(c(a_i1,...,a_ij,...)) = a_ij
         end

Such an assertion is easy to recognize for the compiler. A constant
constructor does not have arguments and therefore does not need a projector.

Equipped with the recognizer expressions and the projector expressions the
compiler is able to transform any general inspect expression of a pseudo
inductive type into the executable expression

    if rec_1(x) then
        exp_n(proj_11(x),...)
    else if rec_2(x) then
        exp_n(proj_21(x),...)
    ...
    else
        exp_n(proj_n1(x),...)


In our example of bounded natural numbers the compiler recognizes in the above
stated axiom

    n.successsor.predecessor = n

the expression `_.predecessor` as a projector expression for the constructor
`successor`.

An inspect expression of the form

    inspect
        n
    case n = 0 then
        exp1
    case n.successor then
        exp2(n)

is translated into the executable expression

    if n = 0 the
       exp1
    else
       exp2(n.predecessor)




## Recursion


Inspect expressions or pattern matching expressions can be used to define
recursive functions.


    (+) (n,k:N): N
        -> inspect
               k
           case 0 then
               n
           case k.successor then
               (n + k).successor


    (*) (n,k:N): N
        -> inspect
               k
           case 0 then
               1
           case k.successor then
               (n * k) + n


As with inductive types the compiler can check the wellfoundedness of the
recursion by checking that in each recursive call at least one of the
arguments is structurally smaller than the corresponding argument in the
origninal call.

These recursive definitions _define_ the functions `+` and `*`. Obviously the
definitions are not very efficient to execute, because the addition of `k` to
another number `n` needs `k` operators.

Any type which inherits `BOUNDED_NATURAL` can define a more efficient
implemention of these functions as long as the definition in the heir is
equivalent to the corresponding recursive definition.

Furthermore we can proof a lot of properties of recursive functions like
associativity, commutativity, distributivity by using the corresponding
induction principle. All these properties are inherited by any type which
inherits the pseudoinductive type.

Note that functions using arbitrarly complex inspect expressions can be
translated to something executable. E.g. the function

    (<=) (n,k:N): BOOLEAN
        -> inspect
               n, k
           case 0, _ then
               true
           case n.successor, k.successor then
               n <= k
           case _, _ then
               false

can be translated into the equivalent function

    (<=) (n,k:N): BOOLEAN
        -> if n = 0 then
               true
           else if n /= 0 and k /= 0 then
               n.predecessor <= k.predecessor
           else
               false

The compiler can verify the soundness of the recursion in the original
definition. The soundness in the translated code is not that easy to check. In
order to check the soundness of the translate function it would be necessary
to prove that `BOUNDED_NATURAL` inherits `WELLFOUNDED` and the recursive
definition would need to define some bound function like

    (<=) (n,k:N): BOOLEAN
        decrease
            n
        ensure
            -> if n = 0 then
                   true
               else if n /= 0 and k /= 0 then
                   n.predecessor <= k.predecessor
               else
                   false
        end

This would require much more machinery because `BOUNDED_NATURAL` cannot
inherit `WELLFOUNDED` unless it has a wellfounded relation `<`. The definition
of `<` however already needs some order relation which had to be defined
independently of `<=` in order to avoid circularity. It is not impossible to
do but not as elegant as the usage of pattern matching with its syntactic
checking of soundness of the recursion.



## History

Pseudoinductive types are not a completely new concept. Philip Wadler
described in [[wadler1987](bibliography.md#wadler1987)] the concept of view
types. View types like pseudoinductive types are types which look like
inductive data types or free data types. A quote from the article:

> Pattern matching and data abstraction are important concepts in designing
  programs but they do not fit well together. Pattern matching depends on
  making public a free data type representation while data abstraction depends
  on hiding the representation. This paper proposes the views mechanism as a
  means of reconciling this conflict. A view allows any type to be viewed as a
  free data type thus combining the clarity of pattern matching with the
  efficiency of data abstraction.

The concept of view types targets mainly at making pattern matching available
to types which are not inductive/free data types. Here we want to make pattern
matching and induction proofs available to types which are not inductive/free
data types.



<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
