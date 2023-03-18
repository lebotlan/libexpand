# Libexpand, a layout solving algorithm

Libexpand is a library used to build and solve 1.5D layout problems.
It can typically compute the layout of tables, that is, the width of each column and height of each row,
according to their content, spacing, alignment, and other constraints (including multi-row, multi-column cells).

The name libexpand comes from the tricky part of the solving algorithm: it handles expansion variables, that is,
spaces that try to expand as much as possible (like \hfill in latex).


## Equations

A problem is a system of non-circular equations of the form

  b1 + cst + k1.h1 + k2.h2 + ... + kn.hn <= b2
  ...

where

 - cst is a numeral constant
 - b1, b2 are position variables (aka box variables, bvars)
 - h1, ... hn are expansion variables (aka hfill variables, hvars)
 - k1 is the coefficient of h1 (and ki is the coef of hi)   k1.h1 is equivalent to h1 + h1 + ...  k1 times if h1 has no minimal and no maximal bound.

Such equations are built using the following OCaml modules, to be found in Libexpand.Constraints:

 - `Hvar` to create hvars
 - `Bvar` to create bvars
 - `Expr` to build expressions (b1 + cst + k1.h1 + k2.h2 + ... + kn.hn)
 - `Dag` to build the system of equations (expr <= b2)
 - `Level` to handle the level (that is the strength) of bvars and hvars


## Layout as equations

An equation captures a 1D constraint, e.g. horizontal constraint *or* vertical constraint (Libexpand computes positions and sizes, it cannot solve Tetris problems).

 - A bvar is a variable holding a position (e.g. the left side or right side of a box).
 - A hvar is an expansion variable that can be used to put space between items, or to left-align, center, or right-align a box content.

Equations may express for example that some items must be vertically aligned (using the same bvar for representing the x-position of the items), or,
as another example, that inner items of a box should be evenly spaced (using hvars between bvars).

Equations can express layout constraints on different layers, which can fork and join at different points. This is why we consider it as a 1.5D layout problem.
It is not a 2D Tetris solver.


### Layout system example: evenly spaced bvars, sharing a 40-wide space:

 b1 + 40 <= b4   ## b4 is at least 40 spaces to the right of b1.
 
 b1 + h1 <= b2   ## b2 and b3 are put between b1 and b4, separated by hvars h1, h2, h3.
 b2 + h2 <= b3
 b3 + h3 <= b4


  b1 <--h1--> b2 <--h2--> b3 <--h3--> b4
     <------------ 40 -------------->


## Meaning of variables 

 - bvars (= x-position) try to be as small as possible
 - hvars (= spacing) try to be as large as possible.

Each bvar or hvar has a *level*. A hvar of level l wins against hvars or bvars of lower levels.

When several hvars of the same level compete, the available space is shared between them.

Hvars can have a minimal and maximal value.

Infinitely expansible hvars are detected and fixed to a predefined value. These are unbounded hvars that are not blocked by any bvar of higher level (this should be considered as a design mistake).



## It must be a DAG

The set of equations must be a dag (there must be no circular dependency between bvars, even legit), e.g.  b1 - 6 <= b2  and b2 + 3 <= b1 is forbidden.


## Usage

Build a dag using the modules in `Constraints`. Solve it using `Solver`.

The dune libraries are:

 - libexpand.constraints
 - libexpand.solve

The Ocaml modules are:

 - Constraints.Dag, Constraints.Bvar, Constraints.Hvar, ...
 - Solve.Solver


## Examples

See test/testlib.ml
You may run `./testlib.exe | less` to understand what the library does.

Also, the dag are exported as graphs in subdirectory Graphs/. Use make in Graphs/ (requires graphviz).


## Table examples

In order to express the constraints of a table layout, use a system for x coordinates, and another one for y coordinates.
Each column (or say, row) is associated to two variables: a_i = x-position of the column left side  b_i = x-position of the column right side.
Here is how to express:

 * Separation between columns
 fixed: b_i + k <= b_(i+1)
 extensible: b_i + h1 <= b_(i+1)

 * Fixed size of a column: a_i + content-width <= b_i

 * Multi-column: a_i + content-width <= b_j

 * Center the content of a column: a_i + h1 + content_width + h1' <= b_i
(the value of h1 and h1' in the solution provides the left and right spacing)

 * Column that expand as much as possible, until a given max width
   a_i + h2 <= b_i  where h2 has a max value.

 * Columns that share some available space. The middle column takes twice the space.

   a1 + h2 <= b1     b1 + 2 <= a2
   a2 + 2.h2 <= b2   b2 + 2 <= a3
   a3 + h2 <= b3

   a1 + 80 <= b3     # Total width



## Test

make
./testlib.exe    should raise no assertion failure

