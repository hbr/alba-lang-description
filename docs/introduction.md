# Introduction

##### Todays Software

Software is an integral part of our life today. It has been a complete success
story in the second half of the 20th century and it continous in the
21st. Software is indispensable in flight control, aerospace, financial world,
telecommunication, research etc. Now smartphones have pentrated the worlds
population. And the world with aoutonomous software controlled cars is not
very far.

Even with todays technology to develop software, bugs are an integral part of
software. Half of the effort to produce software goes into the construction of
the software and the other half goes into finding (hopefully) most of the bugs
in the software before the user runs into some of the remaining
bugs. Unfortunately some severe accidents have happened because of bugs in the
software. Viruses can exploit security bugs in a software to do their bad
business.

There is a popular saying attributed to Edsger Dijkstra: "If debugging is the
activity to remove bugs from the software then programming must be the
activity to introduce them".

And the name _bug_ gives some false associations. It makes the impression as
if bugs creep secretely into the software while the programmer is not looking
at it.

And since software construction is a terribly complex task it seems to be
inevitable that bugs are within any software system since programmers are
human beings which commit errors when confronted with complex tasks.

In the 1960s the term _software crisis_ had been invented to describe the
problem. Since then science has invested a lot of effort to find methods to
build correct software. The so called _formal methods_ said that it is
possible to write software which is correct by construction.


##### The Start of Formal Methods

Robert Floyd and Tony Hoare did some pioneering work in that
area. [Floyd](bibliography.md#floyd1967) observed that by attaching assertions
to edges of a flow graph he can start to reason about software by logic.

Tony Hoare described in his foundational paper ["The axiomatic basis of
computer programming"](bibliography.md#hoare1969) how a language consisting of
assignment, alternative command and while loops can be given rules which can
be used to prove the correctness of programs written in this language.

There are certainly more authors who proposed similar ideas. But Robert Floyd
and Tony Hoare are today the most cited in that are.

So in academia the idea of writing software which is correct by construction
started to gain some momentum.

Edsger Dijkstra used the foundations of Tony Hoare to invent the so called
_weakest precondition calculus_. He and others started to teach this method to
students who learned to write correct algorithms.

David Gries wrote the text book ["The science of
programming"](bibliography.md#gries1981) describing the method in a manner
that it can be used by undergraduate students. In this textbook he expressed
the idea that the method is so powerful that soon more and more programs will
be written by using the method to prove software correct. But we all know that
this had not happened (at least up to now).


##### Drawbacks of the Early Formal Methods

Tony Hoare in his 1967 alredy mentioned that it might be difficult to use the
method in practice.

> However, program proving, certainly at present, will be difficult even for
  programmers of high caliber; and may be applicable only to quite simple
  program designs.

I.e. proving software to be correct requires some skills in logical reasoning
which are not addressed sufficiently in the formation of software developers.

However Dijkstra and Gries proved that the method is teachable with success
even to undergraduate students.

But there is another serious problem. Software of significant size is not
static. There are frequent changes during the development and the maintenance
process. Any change requires that a correctness proof has to be redone to
prove that no bugs creeped into the software. By doing proofs with paper and
pencil, this is practially impossible or more correctly: requires an enormous
effort to do it.

From that experience we can conclude that formal methods can be used
efficiently only if a programming language is used which allows to express the
operational software and its correctness proofs.


##### The Available Tools

There are many tools available on the market to write correctness proofs for
software. The most prominent tools are _Coq_ and _Isabelle_. Others are
_Idris_, _Agda_, etc.

In Coq it is possible to write functional software and prove correctness
properties about that software. The functions defined in Coq can be exported
to a programming language like Ocaml and then compiled to native machine code.

There are some heroic efforts to use Coq to construct real world software. The
most prominent one is [_Compcert_](http://compcert.inria.fr/) which is a
completely verified C compiler.

This proves the claim that it _is_ possible to write formally verified code by
using todays tools. But there is not yet a widespread use of the tools. Why?

- Using a tool like Coq or Isabelle requires skills and both tools have a
  steep learnig curve. Only very few software developer are able and have
  enough patience to acquire these skills.

- Most of the todays tools use constructive logic which is for many software
  developers counterintuitive.

- You cannot generate compiled code directly from the languages. You have to
  go via some intermediate language. But the data types used e.g. in Coq (like
  natural numbers) cannot be transferred easily to the intermediate language.

- The tools usually only support functional and not imperative
  programs. Generating verified imperative code (which is necessary in some
  cases) requires some other intermediate steps which are not easy to do.

Summary: Using formal methods to generate correct software is possible for
real world software projects. But the price to pay in terms of skill
development and intermediate steps is still too high.



##### The Albatross Project

The Albatross programming language with its compiler and verifier tries to
bridge the remaining gaps and shoots at making verification and everydays task
in software development.

Its goal is to make software verification available for the masses.

The Albatross compiler has an integrated proof engine which lifts off the
burden detailed proof steps off the user. The proof engine can do many proof
steps automatically and the developer just has to state the desired
properties. Clearly complex algorithms still require some deep
thinking. However straightforward code should work just out of the box.

The modularity of the language allows to do complicated code and the
corresponding proofs within the implementation part and the user can use the
exported functions and properties in his application code.

There is no intermediate language (at least not visible to the developer) to
compile to. All the glueing of the semantics of the Albatross language and the
semantics of the target language is hidden to the user.

Albatross allow to write functional, imperative and concurrent code within the
same language. Its syntax is similar to mainstream object oriented languages
familiar to todays programmers.

The project is still in its development phase. Therefore not all functionality
is yet available. But the target is to make a programming language which is
general purpose, easy to use and fully verified.


<!--
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
