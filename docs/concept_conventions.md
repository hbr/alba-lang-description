# Conventions


## Comments

    -- A single line comment (the blank after '--' is significant!)

    {: A multiline comment

       It spans over several lines {: and can be nested :}
       until the end of comment marker is detected :}


## Names

All class names and names of class variables are in uppercase. Variable and
function names must be in lower case. Names have to begin with a letter and
followed by an arbitrary sequence of letters, digits or underlines.

Legal classnames and class variable names:

    A
    BOOLEAN
    PARTIAL_ORDER
    CLASS1_BLABLA_345

Legal function and variable names:

    i
    is_lower_bound
    func23_blabla_10




## Calls

The usual mathematical notation is used to call a function. `f(x)` calls the
function `f` with the argument `x`. The convention used in functional
languages to call functions by juxtaposition (i.e. `f x`) is not used in
Alba.

If a function has more than one argument it is called by `f(x,y,...)`. Calls
of the form `f()` are illegal. A constant function doesn't need parentheses,
it is called by writing its pure name.

Functions can be called in object oriented notation as well. The following
expressions are equivalent.

    -- math notation         oo notation

    f(x)                     x.f
    f(f(f(x)))               x.f.f.f
    g(x,y)                   x.g(y)

Whether you use mathematical or object oriented notation is a matter of
taste. Both notations are possible in order to choose the more readable and
expressive one. In my opinion `set.is_empty` is more readable than
`is_empty(set)` and `x.is_least(set)` looks better than `is_least(x,set)`. For
the compiler both are equivalent and can be used arbitrarily.


## Uniqueness of Functions and Overloading

Function names or operator names do not uniquely identify a function. The
operator '+' is used for the addition of two numbers, the union of two sets
etc. Only the function (operator) name together with its signature uniquely
identifies the function.




## Operators

It is possible to define binary and unary operators. Unary operators can be
used only as prefix operators. Postfix operators are not possible.

Operators have precedences and associativities to avoid excessive parentheses
and are grouped into the following precedence categories (highest precedence
first).

- boolean negation: `not`

- exponentiation (right): `^`

- multiplication (left): `*`, `/`, `mod`, `|`

- addition (left): `+`, `-`

- relational (no): `=`, `/=`, `~`, `/~`, `<`, `<=`, `>`, `>=`, `in`, `/in`, `as`

- boolean connectives (left): `and`,  `or`

- implication (right): `==>`


The operators `+`, `-` and `*` can be used as binary and unary
operators. I.e. `a + b`, `+a`, `-a`, `*a`, `+ * a` are syntactically perfect
operator expressions (although not all are meaningful for numbers).

The function addressed by an operator is expressed by the operator in
parentheses. The following two expressions are equivalent.

    a + b

    (+)(a,b)

However the first one is usually preferable because it is better readable.

Parentheses have to be used as well to declare an operator function e.g.

    (+)(a,b:NATURAL): NATURAL ...



## Semicolons and Newlines

Declarations and assertions have to be separated by semicolons. However
explicit semicolons are nearly never used in Alba because the lexical analyzer
recognizes almost all positions where a semicolon might occur. If a newline
appears at a position where a semicolon is possible, the newline serves as an
implicit semicolon.

There is a potential ambiguity with operators which can be binary and
unary. In proofs it is possible to write a sequence of boolean
expressions separated by semicolons or newlines. E.g. the following sequence


    n =  k
       + m
       + 100

    *p + *q = *(p + q)     -- Ambiguity with '*' as unary or binary.

is parsed as `n = k + m * p + *q = *(p + q)` and the compiler reports an error
at the second `=` because `=` is not associative. Parentheses have to be used
to disambiguate the situation.

    n =   k
        + m
        + 100

    (*p) + *q = *(p + q)


> Note: This might be changed in the future by disallowing lines to start with
  a binary operator i.e. that operators which are binary and unary at the
  beginning of a line are treated as unary operators. If used as a binary
  operator they have to be put at the end of line.

Furthermore function calls spanning over several lines must not be separated
before the opening parenthesis. I.e.


    x + y + z + f
    (a,b,...)

is parsed as `x + y + z + f; (a,b,...)` which is probably not the
intention. If `f(a,b,...)` shall be parsed as a function call the following
form is correctly parsed.

    x + y + z + f(a,
                  b,
                  ....)





<!--
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
