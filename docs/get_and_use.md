# Get the Alba Compiler

The compiler is written in `OCaml` and needs an installation of the ocaml
compiler better than version 4.03.0. Furthermore it needs the builder
`ocamlbuild`, the parser generator `menhir` and the ocaml finder `ocamlfind`.

It is best to install the ocaml package manager and the compiler. Both are
available for a lot of platforms and operating systems. Instructions on how to
do the installation can be found on http://ocaml.org.

Having the ocaml package manager and the ocaml compiler you can issue the
commands

    opam install alba

This command installs the Alba compiler and the base library within the
corresponding opam directories.

The command

    alba

should show a short help text on how to use the compiler.




# Use the Alba Compiler

The Albatross compiler compiles packages or more exactly modules within a
package. A package is a directory containing Albatross source files of the
form `<name>.al` and `<name>.ali`.

In order to compile a package it first has to be initialized by issueing the
command

    alba init

within the directory of your source files.

The command

    alba status

shows which files need compilation or recompilation. The compiler has an
automatic dependency management and compiles only packages which need
compilation or recompilation.

The command

    alba compile <module-name>

compiles the module `name` and all modules which are used by the module `name`
and need compilation or recompilation. Note that the module has to be named
without the extension `*.al` or `*.ali`.

The command

    alba compile

compiles the whole package i.e. all modules of the package which need
compilation or recompilation.

In order to force recompilation even if not need you can issue the command

    alba compile -force <module-name>

If the Albatross compiler has been installed via `opam` then the base library
has been installed as well and the Albatross compiler is able to find it. In
case that you have written 2 or more packages and one of the package uses not
only the base library but also some of your own packages, you have to tell the
compiler where to find the used packages.

The command

    alba compile -I <path1> -I <path2> ... <module-name>

compiles the module `name` and searches for used packages in the paths
`path1`, `path2`, ...

If you want to compile a package residing in a different directory than the
current directory you can indicate this to the compiler via the option
`-work-dir`.

    alba compile -work-dir <path-to-source> <module-name>


The command

    alba help <command>

gives a short description on how the use the command `command` e.g. use `alba
help compile` to show the options and arguments for the compile command.


<!--
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
