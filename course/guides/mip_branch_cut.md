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
  display_name: Julia 1.10.3
  language: julia
  name: julia-1.10
---

# Solving Mixed Integer Programs with Branch & Cut

## Mixed Integer Programs (MIPs)

% Should these be MIP/MILP/MIQCP (Quadratic constraints)/MIQP (quadratic objective, linear constraints)
% Maybe explain but just focus on one

Mixed integer programs (MIPs) are those where some of the decision variables are discrete while others are continuous.
These problems may come in a number of varieties, such as mixed integer quadratically constrainted programs (MIQCP), which contain at least one constraint with a quadratic term.
Another variety is mixed integer quadratic programs (MIQPs), whose constraints are linear but the objective function is quadratic.

Here, we will focus on mixed integer linear programs (MILPs), where the objective and all the constraints are linear.
In MILPs (as well as in other MIPs), the discreteness of the problem is often imposed by modeling a choice among a fixed number of options (sometimes as simple as a "yes or no" decision) or having to work with a whole number of resource.
In comparison to purely linear programs of comparable sizes, MIPs are often more difficult to solve.
This is due to the fact that in going from continuous variables to integer ones, the geometry of the feasible region gets more complicated.

## Branch and cut

How can we solve these problems? If all variables were continuous, this would reduce to a linear program, which we know how to solve with the simplex method (or with other methods like the interior point method). Doing so is called a *relaxation*, i.e. approximating a difficult problem by one that is easier to solve. The relaxed LP solution gives an upper bound on the true optimum (assuming one exists), since the feasible set is larger. If the LP solution happens to have integers for the correct variables, then we got lucky and ended up finding the optimum. 

Suppose however that the relaxed solution contains non-integer values for integer variables. What then? One idea is to add a cuts, i.e. linear constraints that exclude the relaxed LP solution but don't exclude any integer points. We then end up with more LPs but with stricker relaxations, which we can continue solving and cutting similarly.

The branching part of this algorithm comes from the fact that as we add cuts and end up with more problems, some of them can be eliminated from consideration without having to solve them. For example, imagine that we solved the LP relaxation of an MILP and the relaxed solution contains {math}`x_1=1.5` where {math}`x_1` is supposed to be an integer. Assuming that these won't eliminate any integer solutions, we may create two LPs by adding constraints {math}`x_1\leq 1` and {math}`x_1\geq 2`. In both these LPs, the original relaxed solution is infeasible. If the former has an integer {math}`x_1` with a better objective function value when the latter has a non-integer {math}`x_1` with a worse objective value, then one can prune the generation of further LPs from the latter. This is because any further LP will add additional constaints, therefore limiting the feasible region which will make the objective function even worse. Since the former already yielded an integer solution with a better objective value, any further LPs will fail to outperform it. 


## Example

### Problem

This example is from {cite}`williams_model_2013` Chapter 12.15.

A number of power stations are committed to meeting the following electricity
load demands over a day:

```{list-table}
:header-rows: 0

* - 12 p.m. to 6 a.m.
  - 15 000 MW
* - 6 a.m. to 9 a.m.
  - 30 000 MW
* - 9 a.m. to 3 p.m.
  - 25 000 MW
* - 3 p.m. to 6 p.m.
  - 40 000 MW
* - 6 p.m. to 12 p.m.
  - 27 000 MW
```

There are three types of generating unit available: 12 of type 1, 10 of type
2 and ﬁve of type 3. Each generator has to work between a minimum and a
maximum level. There is an hourly cost of running each generator at minimum
level. In addition, there is an extra hourly cost for each megawatt at which a
unit is operated above the minimum level. Starting up a generator also involves
a cost. All this information is given in the table below (with costs in £).

In addition to meeting the estimated load demands there must be sufﬁcient
generators working at any time to make it possible to meet an increase in load of
up to 15%. This increase would have to be accomplished by adjusting the output
of generators already operating within their permitted limits.

```{list-table}
:header-rows: 1

* -
  - Minimum level
  - Maximum level
  - Cost per hour at minimum
  - Cost per hour per megawatt above minimum
  - Cost
* - Type 1
  - 850 MW
  - 2000 MW
  - 1000
  - 2
  - 2000
* - Type 2
  - 1250 MW
  - 1750 MW
  - 2600
  - 1.30
  - 1000
* - Type 3
  - 1500 MW
  - 4000 MW
  - 3000
  - 3
  - 500
```

Which generators should be working in which periods of the day to minimise
total cost?

### Modeling

This model has a total of 55 constraints and 30 simple upper bounds. There are 45 variables, of which 30 are general integer variables.

#### Variables

```{math}
n_{ij} = \text{number of generating units of type } i \text{ working in period } j

s_{ij} = \text{number of generators of type } i \text{ started up in period } j

x_{ij} = \text{total output rate from generators of type } i \text{ in period } j
```

{math}`x_{ij}` are continuous, the rest are integer variables.

#### Constraints

Demand must be met in each period:
```{math}
\sum_ i x_{ij} \geq D_j \text{ for all j},
```
where {math}`D_j` is demand given in period {math}`j`.

Output must lie within the limits of the generators working:
```{math}
x_{ij}\geq m_in_{ij} \text{ for all} i \text{ and }j

x_{ij}\leq M_in_{ij} \text{ for all} i \text{ and }j
```
where {math}`m_i` and {math}`M_i` are the given minimum and maximum output levels for generators of type {math}`i`.

The extra guaranteed load requirement must be able to be met witout starting up any more generators:
```{math}
\sum_iM_in_{ij}\geq \frac{115}{100}D_j \text{ for all }j
```

The number of generators started in period {math}`j` must equal the increase in number
```{math}
s_{ij}\geq n_{ij}-n_{ij-1} \text{for all } i \text{ and } j
```
where {math}`n_{ij}` is the number of generators started in period {math}`j` (when {math}`j=1`, period {math}`j-1` is taken as 5).

In addition, all the integer variables have simple upper bounds corresponding to the total number of generators of each type.

#### Objective function (to be minimized)

```{math}
\text{Cost} = \sum_{i,j}C_{ij}(x_{ij}-m_in_{ij})+\sum_{i,j}E_{ij}n_{ij}+\sum_{i,j}F_is_{ij}
```
where {math}`C_{ij}` are costs per hour per megawatt above minimum level multipled by the number of hours in the period; {math}`E_{ij}` are costs per hour for operating at minimum level multipled by the number of hours in the period and {math}`F_i` are start-up costs.

### Solution

First we import packages and define the relevant constants

```{code-cell} julia
:tags: ["remove-output"]

using JuMP
import HiGHS

model = Model(HiGHS.Optimizer)
n_types = [12, 10, 5]
min_level = [850, 1250, 1500]
max_level = [2000, 1750, 4000]
cost_per_hour_at_min = [1000, 2600, 3000]
cost_per_hour_per_mw = [2, 1.30, 3]
cost_per_startup = [2000, 1000, 500]
demand = [15000, 30000, 25000, 40000, 27000]
hours_per_period = [6, 3, 6, 3, 6]
```

Create the model.

```{code-cell} julia
:tags: ["remove-output"]

@variable(model, 0 <= n[i = 1:3, j = 1:5] <= n_types[i], Int)
@variable(model, 0 <= s[i = 1:3, j=1:5] <= n_types[i], Int)
@variable(model, 0 <= x[i = 1:3, j=1:5])

# Demand is met
@constraint(model, demand_met[j in 1:5], sum(x[i,j] for i in 1:3) >= demand[j])
# Power generated is within the limits
@constraint(model, output_lb[i in 1:3, j in 1:5], x[i,j] >= min_level[i]*n[i,j])
@constraint(model, output_ub[i in 1:3, j in 1:5], x[i,j] <= max_level[i]*n[i,j])
# increased load margin is satisfied
@constraint(model, extra_load[j in 1:5], sum(max_level[i]*n[i,j] for i in 1:3) >= 1.15*demand[j] )
# The number of started generators match
@constraint(model, continuity[i in 1:3, j in 2:5], s[i,j] >= n[i,j] - n[i,j-1])
@constraint(model, continuity2[i in 1:3], s[i,1] >= n[i,1] - n[i,5])

@objective(model, Min, 
    sum(cost_per_hour_per_mw[i]*hours_per_period[j]*(x[i,j]-min_level[i]*n[i,j]) for i in 1:3, j in 1:5) 
    + sum(cost_per_hour_at_min[i]*hours_per_period[j]*n[i,j] for i in 1:3, j in 1:5) 
    + sum(cost_per_startup[i]*s[i,j] for i in 1:3, j in 1:5))
```

```{code-cell}
print(model)
```

```{code-cell}
optimize!(model)
```

```{code-cell}
is_solved_and_feasible(model)
```

```{code-cell}
print(objective_value(model))
print(value.(n), "\n")
print(value.(s), "\n")
print(value.(x))
```