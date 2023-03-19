# Libexpand, a layout solving algorithm

[Libexpand](https://github.com/lebotlan/libexpand) is a library used to build and solve 1.5D layout problems.
It can typically compute the layout of tables, that is, the width of each column and height of each row,
according to their content, spacing, alignment, and other constraints (including multi-row, multi-column cells).

The name **libexpand** comes from the tricky part of the solving algorithm: it handles *expansion* variables, that is,
spaces that try to expand as much as possible (like \hfill in latex).

## Documentation



## Overview

Layout constraints are collected in a set of inequations, which are then solved by libexpand. Horizontal
and vertical constraints are separated.

### Example 1: share a 23-wide horizontal space between 4 elements.

Consider this example:

```latex
|This space width is 22|   % Two bars separated by a 22-wide space.
|                      |
|<h1><h2><h3><h4>      |   % Four elements that try to share the space.

b1                     b2
```

- Variable b1 holds the position of the left bar.
- Variable b2 holds the position of the right bar.

- b1, b2 are *position* variables - they tend to be as small as possible
- h1, h2, h3, h4 are *expansion* variables - they tend to be as large as possible, but cannot increase b2's value.

The corresponding system of inequations:

```latex
 b1 + 22 < b2                   % There is at least a 22-wide space between b1 and b2. 
 b1 + h1 + h2 + h3 + h4 < b2    % The space between b1 and b2 is occupied by h1, h2, h3, h4.
```

The solution, as computed by libexpand, provides integer values for all variables b1, b2, h1, h2, h3, and h4. Since b1 is unconstrained, its value is 1.
 
A solution:
```latex
  b1 = 1  b2 = 23
  h1 = 6  h2 = 5  h3 = 6  h4 = 5
  
  |< h1 ><h2 >< h3 ><h4 >|
```

The 22-wide space is shared between h1, h2, h3 and h4 as evenly as possible. Admissible solutions are such that h1,h2,h3 and h4 have a distance of at most 1, and a sum of 22.

TODO: See [the full example in OCaml](Example1.md)


### Example 2: alignment in a table's cells.

```latex
|This line is 32 characters long.|   % Line 1
|                                |
|            Centered            |   % Line 3
|Left                            |   % Line 4
|                           Right|   % Line 5

b1                                b2
```

The constraints associated to the table example above are:
- Line 1 : `b1 + 32 < b2`
- Line 3 : `b1 + h1 + 8 + h2 < b2`  where h1,h2 are expansion variables used to add space around the centered text.
- Line 4 : `b1 + 4 + h3 < b2`   the expansion variable takes the space available after the text.
- Line 5 : `b1 + h4 + 5 < b2`   the expansion variable takes the space available before the text(*).

(*) the equation itself does not care much about left or right: equations for Line 4 and Line 5 are actually similar.

The solution computed by libexpand provided suitable values for b1,b2,h1,h2,h3,h4.
 
 ```latex
  b1 = 1  b2 = 33
  h1 = 12  h2 = 12  h3 = 28  h4 = 27
```
 
 TODO: See [the full example in OCaml](Example2.md)

### What else?

- Expansion variables can have a **minimal** and **maximal** size.
- Position variable and expansion variables have a **level**. A variable can be pushed by variables with a level that is higher or equal.
- Constraints can be **deeply nested** - it is possible to model tables in tables in a table.
- The system is expressive enough to model multi-column cells, or say, alignment cosntraints between elements located in different tables.


## Equations, variables, and modeling

Read [Equations.md](Equations.md)


## Usage

Build a dag using the modules in `Constraints`. Then solve it using `Solver`.

The dune libraries are:

 - libexpand.constraints
 - libexpand.solve

The Ocaml modules are:

 - Constraints.Dag, Constraints.Bvar, Constraints.Hvar, ...
 - Solve.Solver




## Examples

See test/testlib.ml.
You may run `./testlib.exe | less` to understand what the library does.

Also, the dag are exported as graphs in subdirectory Graphs/. Use make in Graphs/ (requires graphviz).





## Test

make
./testlib.exe    should raise no assertion failure


## How does it work?

The algorithm is iterative: it computes a solution until a fixpoint is reached. 
Yet, constraint propagation is not too dumb, so that a solution is usually found in one or two iterations on
most examples, even on tricky ones.

Details are in `solve_phi.mli`


