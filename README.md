# The Zarith library

## OVERVIEW

This library implements arithmetic and logical operations over
arbitrary-precision integers.  

The module is simply named `Z`.  Its interface is similar to that of
the `Int32`, `Int64` and `Nativeint` modules from the OCaml standard
library, with some additional functions.  See the file `z.mlip` for
documentation.

The implementation uses GMP (the GNU Multiple Precision arithmetic
library) to compute over big integers. 
However, small integers are represented as unboxed Caml integers, to save 
space and improve performance. Big integers are allocated in the Caml heap, 
bypassing GMP's memory management and achieving better GC behavior than e.g. 
the MLGMP library.
Computations on small integers use a special, faster path (coded in assembly
for some platforms and functions) eschewing calls to GMP, while computations
on large intergers use the low-level MPN functions from GMP.

Arbitrary-precision integers can be compared correctly using OCaml's 
polymorphic comparison operators (`=`, `<`, `>`, etc.). 
This requires OCaml version 3.12.1 or later, though.

Additional features include:
* a module `Q` for rationals, built on top of `Z` (see `q.mli`)
* a compatibility layer `Big_int_Z` that implements the same API as Big_int from the legacy `Num` library, but uses `Z` internally

### Literals

The `zarith-ppx` package provides literals 
for arbitrary-precision integers and rationals.
Here are some examples:

```ocaml
# 1z ;;
- : Z.t = 1
# 1.234e40z ;;
- : Z.t = 12340000000000000000000000000000000000000
# 0x18z ;;
- : Z.t = 24
# 1.2q ;;
- : Q.t = 6/5
# 2.45e10q ;;
- : Q.t = 24500000000
# 2.45e-5q ;;
- : Q.t = 49/2000000
# -0O123q ;;
- : Q.t = -83
```

It also raise errors for incorrect literals:
```ocaml
# 1.234e2z ;;
Error: This literal does not fit in an integer.
```

## REQUIREMENTS

* OCaml, preferably version 3.12.1 or later.  (Earlier versions are usable but generic comparisons will misbehave.)
* Either the GMP library or the MPIR library, including development files.
* GCC or Clang or a gcc-compatible C compiler and assembler (other compilers may work).
* The Perl programming language.
* The Findlib package manager (optional, recommended).


## INSTALLATION

1) First, run the "configure" script by typing:
```
   ./configure
```
The `configure` script has a few options. Use the `-help` option to get a
list and short description of each option.

2) It creates a Makefile, which can be invoked by:
```
   make
```
This builds native and bytecode versions of the library.

3) The libraries are installed by typing:
```
   make install
```
or, if you install to a system location but are not an administrator
```
   sudo make install
```
If Findlib is detected, it is used to install files. 
Otherwise, the files are copied to a `zarith/` subdirectory of the directory 
given by `ocamlc -where`.

The libraries are named `zarith.cmxa` and `zarith.cma`, and the Findlib module
is named `zarith`. 

Compiling and linking with the library requires passing the `-I +zarith`
option to `ocamlc` / `ocamlopt`, or the `-package zarith` option to `ocamlfind`.

4) (optional, recommended) Test programs are built and run by the additional command
```
  make tests
```
(but these are  not installed).

5) (optional) HTML API documentation is built (using `ocamldoc`) by the additional command
```
  make doc
```

## LICENSE

This Library is distributed under the terms of the GNU Library General
Public License version 2, with a special exception allowing unconstrained 
static linking. 
See LICENSE file for details.


## AUTHORS

* Antoine Miné, Université Pierre et Marie Curie, formerly ENS Paris.
* Xavier Leroy, INRIA Paris-Rocquencourt.
* Pascal Cuoq, CEA LIST.


## COPYRIGHT

Copyright (c) 2010-2011 Antoine Miné, Abstraction project.
Abstraction is part of the LIENS (Laboratoire d'Informatique de l'ENS),
a joint laboratory by:
CNRS (Centre national de la recherche scientifique, France),
ENS (École normale supérieure, Paris, France),
INRIA Rocquencourt (Institut national de recherche en informatique, France).


## CONTENTS

Source files        | Description
--------------------|-----------------------------------------
  configure         | configuration script
  caml_z.c          | C implementation of all functions
  caml_z_*.S        | asm implementation for a few functions
  z_pp.pl           | script to generate z.ml[i] from z.ml[i]p
  z.ml[i]p          | templates used to generate z.ml[i]p
  big_int_z.ml[i]   | wrapper to provide a Big_int compatible API to Z
  q.ml[i]           | rational library, pure OCaml on top of Z
  projet.mak        | builds Z, Q and the tests
  tests/            | simple regression tests and benchmarks

Note: `z_pp.pl` simply scans the asm file (if any) to see which functions have
an asm implementation. It then fixes the external statements in .mlp and 
.mlip accordingly.
The argument to `z_pp.pl` is the suffix `*` of the `caml_z_*.S` to use (guessed by configure).
