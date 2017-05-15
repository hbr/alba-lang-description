# Abstract Classes

Abstract class can be used to describe some general behaviour which other
classes can inherit. In order to inherit an abstract type the inheriting class
has to implement the basic functions and the axioms.




## Declaration

##### Simple Abstract Classes

An abstract class is declared by prefixing the keyword `deferred` in front of
the class. E.g.

     deferred class PARTIAL_ORDER

declares `PARTIAL_ORDER` as an abstract class.

Abstract classes cannot be used to declare types of variables. Instead of an
abstract class a class variable based on the abstract class has to be used to
declare variable types.

With

    PO: PARTIAL_ORDER

we declare the class variable `PO` which represents any class which inherits
the class `PARTIAL_ORDER`.

It is possible to declare a class variable together with the declartion of an
abstract class as in

    deferred class PO: PARTIAL_ORDER

In order to be useful an abstract class needs some abstract
functions. Abstract functions are functions with a deferred definition.

    (<=) (a,b:PO): BOOLEAN
        deferred end

The deferred function `<=` requires every descendant of `PARTIAL_ORDER` to
have a corresponding function.

Note that the how the class variable `PO` is used to declare the types of the
variables `a` and `b`.


Based on the abstract functions we can declare arbitrary other functions.

    is_lower_bound (a:PO, p:{PO}): ghost BOOLEAN
            -- Is 'a' a lower bound for all elements of the set 'p'?
        -> all(x) x in p ==> a <= x

Since abstract functions have no definition we need some other way to describe
their behaviour. Abstract properties are used to achieve that. Abstract
properties are assertions which have a deferred proof. A class inheriting
`PARTIAL_ORDER` either has to provide a proof of these properties or redeclare
them as deferred (in case that the heir is an abstract class as well).

In a partial order the function `<=` has the following properties.

    all(a,b,c:PO)
        ensure
            a <= a                             -- reflexivity
            a <= b ==> b <= c ==> a <= c       -- transitivity
            a <= b ==> b <= a ==> a = b        -- antisymmetry
        deferred
        end

These are the mathematical requirements for a relation to represent a partial
order. It has to be reflexive, transitive and antisymmetric.

We can regard the abstract properties as axioms and prove some other
properties based on these axioms.

    all(a,b:PO, p:{PO})
        require
            a <= b
            b.is_lower_bound(p)
        ensure
            a.is_lower_bound(p)
        end

This property is proved by the proof engine. The proof has to expand the
definition of `is_lower_bound` and to apply of the transitivity law.


##### Deferred Classes with Formal Generics

An abstract class can have formal generics. It is possible to declare an
abstract list as a sequence of elements of an arbitrary type.

    A: ANY

    deferred class AL: ABSTRACT_LIST[A]

    []: AL[A]
            -- The empty list
        deferred
        end

    (^) (x:A, xs:AL[A]): AL[A]
            -- The element 'x' prepended to the list.
        deferred
        end

Abstract functions can have preconditions.

     head (xs:AL[A]): A
         require
             xs /= []
         deferred
         end

     tail (xs:AL[A]): A
         require
             xs /= []
         deferred
         end

In order to define the behaviour of the abstract functions we need some
abstract properties.

    all(x,y:A, xs,ys:AL[A])
        ensure
            [] = x ^ xs ==> false             -- inversion
            x ^ xs = y ^ ys ==> x = y         -- injectivity
            x ^ xs = y ^ ys ==> xs = ys       --    "

            (x ^ xs).head = x
            (x ^ xs).tail = xs
        deferred
        end

Now we could start to prove properties based on the functions and the abstract
properties.


##### Deferred Inductive Classes

The above declared abstract class `ABSTRACT_LIST` seems to be very similar to
the definition of the class `LIST` as an inductive class (as described in the
chapter [Inductive Classes](class_inductive.md)). Wouldn't it be great to have
the power of inductive classes (like recursive functions, the possibilty to do
induction proofs) available within abstract types. And yes, Albatross allows
to declare abstract inductive classes.


    deferred class
        AL: ABSTRACT_LIST[A]
    create
        []
        (^) (x:A, xs:AL[A])
    end

With that declaration the compiler generates an induction law and the
inversion and injectivity laws of equality as abstract properties. Having this
it is possible to use pattern matching to define functions and even recursive
functions like the following ones

    head (xs:AL[A]): A
        require
            xs as (_ ^ _)
        ensure
            -> inspect
                   xs
               case x ^ _ then
                   x
        end

    (+) (a,b: AL[A]): AL[A]
        -> inspect
               a
           case [] then
               b
           case hd ^ tl then
               hd ^ (tl + b)

We can even prove the associativity of concatenation by an induction proof.

    all(a,b,c: AL[A])
        ensure
            a + b + c = a + (b + c)
        inspect
            a
        end


##### General Declaration Pattern

A declaration of an abstract class is done by declaring

- a deferred class

- a class variable which represents classes based on the abstract class

- one or more deferred functions

- some other functions based on the deferred functions

- some abstract properties which describe the behaviour of the functions

- some properties which can be proved based on the abstract properties.



##### Rules

- The abstract class, the abstract functions and the abstract properties have
  to be declared within the same module.

- If an abstract class is exported, all the abstract functions and properties
  must be exported as well.

- An abstract function and an abstract property must involve exactly one class
  variable representing a type based on an abstract class declared in the same
  module. I.e. an abstract function and an abstract property must be owned by
  one abstract class.

The following declaration is illegal.

    PO1: PARTIAL_ORDER
    PO2: PARTIAL_ORDER

    some_function (a:PO1, b:PO2): SOME_TYPE
            -- ILLEGAL declaration, two formal generics!!
        deferred
        end


##### Inheritance from ANY

With the declaration of an abstract class the compiler declares automatically
an abstract equality function and lets the newly declared abstract class
inherit `ANY`. I.e. the following declarations are added automatically by the
compiler.

    -- Automatically added by the compiler
    (=) (a,b:PO): BOOLEAN
        deferred
        end

    deferred class
        PARTIAL_ORDER
    inherit
        ANY
    end

With these implicit declarations the class has an equality function which is
reflexive, symmetric and transitive and which is guaranteed to be a leibniz
equality which allows substitution of equals for equals.

The next chapter explains in detail what inheritance means.



## Inheritance

Every abstract class can be inherited by another class as long as the other
type implements all abstract functions, proves all abstract properties and has
only consistent redefinitions of concrete functions.

The declaration of an inheritance relation has the following general form.

    class
         C[A1,A2,...]         -- class 'C' is the inheriting class
    inherit
         B1[...]              -- class 'C' inherits type 'B1'
         B2[...]              -- class 'C' inherits type 'B2'
             rename
                 f as g,      -- and renames some functions of 'B2'
                 ...
             end
         ...
    end

where `A1`, `A2`, ... are formal generics and the actual generics (if any) of
the base classes only involve these formal generics and no other class
variables.

More than one inheritance relation can be declared by one inheritance
declaration although usually only one inheritance relation is declared.

The compiler does some checks before accepting that class `C` inherits class
`B`.

- `B` must be an abstract class.

- No cycles: The inheritance graph must be acyclic i.e. `C` cannot inherit `B`
  if `B` already inherits `C` directly or indirectly.

- `C` must define all abstract functions of `B`: If `B` has an abstract
  function `f` then `C` must have a function `f` (or with a corresponding name
  according to the rename clause) which is a variant of the function `f` of
  the class `B`. If `C` is an abstract class then the variant of the function
  `f` can be abstract as well.

- `C` must have variants of the abstract properties of `B`

- `C` can have only consistent variants (i.e. consistent redefinitions) of
    concrete functions of `B`

After having inherited from an abstract class no more abstract functions and
abstract properties can be declared for the abstract class.

In the following we demonstrate inheritance with a simple example. We define
the abstract class of a semilattice. Mathematically a (meet) semilattice is a
structure with one algebraic operation (here `*`) which is idempotent,
symmetric and associative.

    deferred class SL: SEMILATTICE

    (*) (a,b:SL): SL  deferred end

    all(a,b,c:SL)
        ensure
            a * a = a                   -- idempotent
            a * b = b * a               -- symmetric
            a * b * c = a * (b * c)     -- associative
        deferred
        end

We want to prove that a semilattice has an order structure i.e. we can define
an order relation which is reflexive, antisymmetric and transitive. First we
define the order relation based on the algebraic operation of the semilattice.

    (<=) (a,b:SL): BOOLEAN
        -> a = a * b

Evidently the relation `<=` is reflexive

    all(a:SL)
        ensure
            a <= a
        end

which is proved by expanding the definition of `<=` and using idempotence. The
property of antisymmetry needs a short explicit proof.

    all(a,b:SL)
        require
            a <= b     -- (1)
            b <= a     -- (2)
        ensure
            a = b
        via [ a
            , a * b    -- (1), def '<='
            , b * a    -- commutativity
            , b        -- (2), def '<='
            ]
        end

The same applies to the transitivity of `<=`.

    all(a,b,c:SL)
        require
            a <= b    -- (1)
            b <= c    -- (2)
        ensure
            a <= c
        assert
            ensure
                a = a * c
            via [ a
                , a * b           -- (1), def '<='
                , a * (b * c)     -- (2), def '<='
                , a * b * c       -- associativity
                , a * c           -- (1), def '<='
                ]
            end
        end

Up to now we have defined a relation `<=` in the class `SEMILATTICE` which is
reflexive, antisymmetric and transitive. All requirements of a partial order
are satisfied. Therefore we are now able make the inheritance declaration
which the compiler accepts.

    deferred class
        SEMILATTICE
    inherit
        PARTIAL_ORDER
    end



## Function Redefinitions

Descendants of an abstract class can have their own definitions of functions
defined in the abstract class.

However this cannot be arbitrary since the abstract class has used its own
definitions to prove properties. Therefore the redefinition of a function in
the inheriting class has to be consistent with the definition of the function
in the abstract class.

In order to prove consistency the compiler transforms the specification of
functions into the context of the descendant class. E.g. in the class
`PARTIAL_ORDER` we have defined the function

    is_lower_bound (a:PO, p:{PO}): ghost BOOLEAN
            -- Is 'a' a lower bound for all elements of the set 'p'?
        -> all(x) x in p ==> a <= x

This function has the specification

    all(a:PO, p:{PO})
        a.is_lower_bound(p) = (all(x) x in p ==> a <= x)

This specification can be transformed into the context e.g. of a
`SEMILATTICE` by substituting the class variable `PO` by the class variable
`SL`.

    all(a:SL, p:{SL})
        a.is_lower_bound(p) = (all(x) x in p ==> a <= x)

If `SEMILATTICE` defines an own version of `is_lower_bound` the redefined
version has to satisfy this property as a consistency condition. The compiler
tries to prove the transformed specification of `is_lower_bound` and only if
it succeeds the redefinition is accepted. Otherwise an error is flagged and
the redefinition is not accepted.



## Rename

In the declaration of an inheritance relation functions can be renamed.

    class
        C[A1,...]
    inherit
        B[...]
            rename
                f as g,
                ...
            end
        ...
    end

A rename clause `f as g` is valid only if `f` is an inheritable feature of `B`
(i.e. if it has exactly one class variable based on `B`).

If `f` is an abstract function then the class `C` must have a corresponding
function named `g`. If `f` is a concrete function then there might be a
variant with name `g` in the class `C` which has to be consistent with `f` in
the sense of the previous chapter.

If there does not exist a variant of a concrete function `f` then the compiler
makes `f` renamed to `g` available in the descendant.

Furthermore the compiler makes all properties involving the function `f`
available in the descendant class `C` where the function `f` is renamed into
the function `g`.



<!--
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
