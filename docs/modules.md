# Module and Package System

## Modules and Packages

A module is a compilation unit. It consists of an implementation file `.al`
and an interface file `.ali`.

A module file has the structure

    use
        used_module_1
        used_module_2
        ...
    end

    declaration_1

    declaration_2

    ...



A package/library is a collection of modules residing in one directory. A
package directory has to be initialized with the command `alba init` to become
an Albatross package.

Each package which is not the base library has to use the base library
directly or indirectly. Beside the base library it can use other libraries.

Modules of the same package are used without qualification. Modules of other
packages must be qualified with their package name. E.g. in order to use the
module `predicate_logic` of the base library, the usage block of the using
module must have the form

    use
        alba.base.predicate_logic
        ...
    end

where `alba.base` is the package name and `predicate_logic` is the module name
and the module `predicate_logic` consists of the files `predicate_logic.al`
and `predicate_logic.ali` in the directory of the package `alba.base`. The
user of the module has only access to the declarations of the interface file
regardless whether the using module resides in the same package or in a
different package.


The Albatross compiler needs a way to find the location of the used
packages. There are two ways to tell the compiler where packages can be
found. Either via command line options or via an environment variable. The
command

    alba -I path1 -I path2 -Ipath3 ... 

tells the compiler to search in the directories `path1`, `path2`, `path3`,
... for Albatross libraries. The paths are searched in the given sequence. If
a used package is not found on one of the paths then the directories pointed to
by the environment variable `ALBA_LIB_PATH` are searched. The environment
variable `ALBA_LIB_PATH` can contain a list of directories separated by `:`.

The usage graph of packages must be cycle free. Furthermore the usage graph of
modules must be cycle free as well.



## Dependency Tracking

The command `alba compile m` compiles the module `m` i.e. it looks for the
files `m.al` and `m.ali` in the working directory which must have been
initialized as an Albatross directory.

The compiler analyzes dependencies within a package. I.e. if the module `m`
uses the module `n` of the same package and the module `n` has been modified
since the last compilation the compiler compiles module `n` before compiling
module `m`.

This guarantees that a module is compiled only if all directly or indirectly
used modules of the same package have been compiled successfully. Therefore
there is no need for makefiles.

Issuing the command

    alba compile

compiles all modules of the current package which need recompilation.

The command

    alba status

lists all modules of the current package which need recompilation.






## Define Before Use

Albatross has a strict _define before use_ policy. Any function, type, theorem
... cannot be used before being defined properly.



## Implementation and Interface

Each module consists of an implementation and an interface file. The interface
file contains just the declarations of the implementation file which are
exported to other modules.

No implementations are given in the interface file. I.e. all theorems are just
listed without any proof information. Functions can be declared in the
interface file without any definition or with definition term or with the
corresponding properties of the function result (details see chapter
[Functions](functions.md)).






<!---
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
