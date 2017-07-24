# Builtin Types and Functions

The builtin types and the corresponding basic functions are all declared in
the module `core` of the basic library `alba.base`.

All modules must used the module `core` either directly or
indirectly. Therefore all builtin types and their corresponding primitive
functions and properties are potentially available.

The module `core` is minimalistic in the sense that it just declares what is
absolutely necessary to use builtin types and reason about them.

E.g. the module `core` declares the type `BOOLEAN` with all boolean
connectives like `not`, `and`, `or` and `==>` and some properties of these
connectives. In order to use booleans effectively in application modules more
properties are needed than the ones declared in `core`. The basic library has
a module `boolean` which declares more assertions than the minimalistic ones
to do boolean logic.





<!--
Local Variables:
mode: outline
coding: iso-latin-1
outline-regexp: "#+"
End:
-->
