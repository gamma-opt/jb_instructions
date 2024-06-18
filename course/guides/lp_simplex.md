---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.2
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Linear Optimization and the Simplex Method


## Linear Optimization

TODO: Write this section better and in more detail

I'm assuming LP is explained in the lectures already but here is a short reminder.

We want to solve problems that are of the form
```{math}
:label: lp_form

\maxi c^\intercal x \\
\stf Ax\leq b \\
x\geq 0
```
where $x\in\reals^n$ are the decision variables, $A\in\reals^{m\times n}$ and $b\in\reals^m$ specify the constraints, and $c\in\reals^n$ gives the linear objective.
LPs of other forms can be transformed into this form as well.
In the lectures, you probably learnt that an LP has a (bounded) optimal solution, then there exists an extreme point of the feasible region which is optimal.

Since at least one vertex is guaranteed to be the optimum, one can just go through every edge to find it.
The simplex algorithm does so, in an efficient way (most of the time).

## The Simplex Method

In order to describe the basic simplex algorithm, consider an objective function of the form 
```{math}
:label: lp_obj_form

a-c_1x_1-\dots-c_nx_n.
```
If we know that all {math}`x_i\geq 0`, then we can maximize it by setting all variables to $0$, giving us the optimal value as {math}`a`.
Thus the problem of optimizing {eqref}`lp_form` becomes a problem of rewriting it in the form of {eqref}`lp_obj_form`.

To exemplify this, consider the following LP

```{math}
:label: lp_example

\maxi 40x_1+60x_2 \\
\stf 2x_1+x_2\leq 7 \\
x_1+x_2\leq 4 \\
x_1+3x_2\leq 9 \\
x_1,x_2\geq 0.
```

The first step is to rewrite {eqref}`lp_example` in the canonical form, where
- inequality constraints are converted to equality constraints by adding non-negative slack variables,
- all variables are non-negative
- the constraints are written so that the slack variables are left on one side.
%TODO complete

```{math}
\maxi 40x_1+60x_2 \\
\stf x_3=7-2x_1-x_2 \\
x_4=4-x_1-x_2 \\
x_5=9-x_1-3x_2 \\
x_1,x_2,x_3,x_4,x_5\geq 0.
```

In the above, the set of (lone) variables in the LHS of the constraints (also called _basic_ variables) correspond to the vertices of the polyhedral that is the feasible region.
By substituting the _nonbasic_ variables in the LP with the basic ones, one is in effect iterating through the vertices of the feasible region, which will ultimately yield the optimal solution.

Actually doing so involves making decisions, for example which variable in the objective function to replace, and which constraint to rewrite it with.
For the purposes of this example, we'll first choose {math}`x_1`, and rewrite it using the **most limiting constraint**, which we obtain by posing the question "What is the largest value {math}`x_1` can have among the constraints?"

```{list-table}
:header-rows: 0

* - {math}`x_3=7-2x_1-x_2`
  - {math}`x_1\leq 3.5`
* - {math}`x_4=4-x_1-x_2 `
  - {math}`x_1\leq 4`
* - {math}`x_5=9-x_1-3x_2`
  - {math}`x_1\leq 9`
```

We can see that the most limiting constraint is {math}`x_3=7-2x_1-x_2`, so we solve for {math}`x_1` to get {math}`x_1=3.5-0.5x_2-0.5x_3`.
Substituting this to all {math}`x_1`s yields

```{math}
\maxi 40(3.5-0.5x_2-0.5x_3)+60x_2 \\
\stf x_1=3.5-0.5x_2-0.5x_3\\
x_4=4-(3.5-0.5x_2-0.5x_3)-x_2 \\
x_5=9-(3.5-0.5x_2-0.5x_3)-3x_2 \\
x_1,x_2,x_3,x_4,x_5\geq 0
```

```{math}
\maxi 140+40x_2-20x_3 \\
\stf x_1=3.5-0.5x_2-0.5x_3\\
x_4=0.5-0.5x_2 +0.5x_3\\
x_5=5.5-2.5x_2+0.5x_3 \\
x_1,x_2,x_3,x_4,x_5\geq 0
```

This completes a single iteration of the algorithm.
By repeating until no positive variable is left in the objective function, one obtains the entire algorithm.

## Visualization

The visualization below is obtained via [GILP](https://gilp.henryrobbins.com/en/latest/index.html).
You can hover over the vertices to see additional information, such as the value of the objective function at that point (labeled **Obj**), or the basic variables associated with the vertex (labeled **B**).

```{code-cell}
import numpy as np

from gilp.simplex import LP
from gilp.visualize import simplex_visual

A = np.array([[2,1],
              [1,1],
              [1,3]])
b = np.array([7,4,9])
c = np.array([40, 60])

lp = LP(A,b,c)
simplex_visual(lp).show(renderer="notebook")
```