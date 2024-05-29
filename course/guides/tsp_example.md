---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.1
kernelspec:
  display_name: Julia 1.10.3
  language: julia
  name: julia-1.10
mystnb:
    execution_timeout: 300
---

# Animations and Interaction with Traveling Salesperson Problem

In an attempt to try using a number of genetic algorithms packages I've been seeing, and also get more comfortable with WGLMakie and Bonito, I've decided to do some TSP based visualisations.
In this guide, I'll go through what I tried and the end results, in the hope that it may be a helpful reference for the future.

## Generating the problem

```{code-cell}
using Random, Metaheuristics, WGLMakie, Bonito
Page(exportable=true, offline=true)
```

Here is a quick way to generate some data for which one can try to find the best (i.e. lowest cost/shortest) Hamiltonian cycle. This was taken from this [JuMP tutorial](https://jump.dev/JuMP.jl/stable/tutorials/algorithms/tsp_lazy_constraints/).

```{code-cell}
:tags: ["remove-output"]

function generate_distance_matrix(n; random_seed = 1)
    rng = Random.MersenneTwister(random_seed)
    X_coord = 100 * rand(rng, n)
    Y_coord = 100 * rand(rng, n)
    d = [sqrt((X_coord[i] - X_coord[j])^2 + (Y_coord[i] - Y_coord[j])^2) for i in 1:n, j in 1:n]
    return d, X_coord, Y_coord
end
```

Let's pick a specific $n$ and observe what the graph looks like.

```{code-cell}
n = 40
d, X_coord, Y_coord = generate_distance_matrix(n)

scatter(X_coord, Y_coord)
```

## Solving the problem

When it comes to evolutionary algorithms, I have encountered 5 Julia packages: [`Metaheuristics.jl`](https://jmejia8.github.io/Metaheuristics.jl/stable/), [`Evolutionary.jl`](https://wildart.github.io/Evolutionary.jl/stable/), [`Optim.jl`](https://julianlsolvers.github.io/Optim.jl/stable/), [`GeneticAlgorithms.jl`](https://github.com/WestleyArgentum/GeneticAlgorithms.jl), and [`GAFramework.jl`](https://github.com/vvjn/GAFramework.jl).
This is not meant to be an exhaustive list and thus may be missing other packages.

Of these 5, the last two are specifically focused on GAs and the third doesn't have GAs but other algorithms.
I've decided to try out the first two only, in part because they are both wrapped by [`Optimization.jl`](https://docs.sciml.ai/Optimization/stable/), which I thought may make plugging one out and the other in simpler.

### Evolutionary.jl

Having decided to try `Evolutionary.jl` first for no reason, I had to formulate and feed the problem in a form acceptable by the package.
Here, I ran into a problem: its documentation states that `Evolutionary` accepts constraints of the form
```{math}
l_x\leq x\leq u_x

l_c \leq c(x)\leq u_c
```
(though presumably, those should be $x_j$ instead of $x$.)
This on its own is not a problem, for example one can use the Dantzig-Fulkerson-Johnson formulation of TSP as an integer linear program, which would satisfy this form.
But I couldn't figure out the `Evolutionary` interface to supply the function $c$.

Since it has a wrapper, I tried using `Optimization.jl` as an interface, but I still kept running into issues, and thus gave up.

### Metaheuristics.jl

This ended up being much easier, primarily due to baked in support for `SearchSpaces.jl` (by the same author) and the provided `PermutationSpace` search space, offering a much more intuitive formulation of the TSP problem.

```{code-cell}
:tags: ["remove-output"]

bounds = PermutationSpace(n)
```

The permutation representation gives the order of visiting various locations, and automatically incorporates constraints such as visiting every city and only once and without subloops.
This way, the objective function also becomes very easy to define.
```{code-cell}
:tags: ["remove-output"]

function tsp_distance(x)
    total = 0
    for i in 1:(n-1)
        total += d[x[i],x[i+1]]
    end
    total += d[x[n],x[1]]
    return total
end
```

One can then specify settings like iteration limit, tolerances and more.
```{code-cell}
options = Options(iterations=1000, seed=1)
```

Then we need to pick the algorithm to use.
I'll use the genetic algorithm implementation but the package offers others too, such as particle swarm.
```{code-cell}
ga = GA(;p_mutation = 0.5,
        p_crossover = 0.9,
        initializer = RandomPermutation(N=100), 
        crossover=OrderCrossover(), 
        mutation=SlightMutation(),
        options=options);
```

The last step is to run the optimization.
```{code-cell}
result = optimize(tsp_distance, bounds, ga)
```

With this at hand, we can access the best result obtained and see what it looks like.
```{code-cell}
sol = minimizer(result)
fig, ax, splot = scatter(X_coord, Y_coord)
lines!(ax, X_coord[sol], Y_coord[sol])  # reorder vector using permutation

endpoints = sol[[1,n]]
lines!(ax, X_coord[endpoints], Y_coord[endpoints])  # connect the cycle
fig
```

## Animations

Animations in `Makie` work by creating plots where every changing element is an Observable, and changing the values of these Observables to obtain different frames.

Suppose we want to animate the above plot to show the solutions obtained in each iteration. 
One fundamental element that is changing across iterations is the best solution for a given generation.
We will keep track of it with an Observable.
```{code-cell}
perm = Observable(collect(1:n))
```
Here, `collect(1:n)` acts as a starting value.
The type of Observables are fixed when they are created, so it can be important to specify that, but in our case, every permutation is represented as `Vector{Int32}`, so we are fine.

We can also keep track of the iteration to display in each frame.
```{code-cell}
it = Observable(0)
```

Similar to the previous plots on this guide, we first plot the scatter plot.
```{code-cell}
fig, ax, splot = scatter(X_coord, Y_coord, axis=(title= @lift("t=$($it)"), ))
```
Here, the `@lift` macro creates a new Observable that depends on an already existing one.
This way, when we change the value of `it` for example to $1$, `Makie` will now that the title needs to become `t=1`.
You can read more about this [here](https://docs.makie.org/stable/explanations/nodes/), but the important part is that they are now linked, and that the syntax requires the Observable to be prepended with a dollar sign (which is inside a `$(..)` since we are also doing string formatting).

To keep things a bit tidier, I define a couple more things as linked Observables.
```{code-cell}
# Lines reordered according to the new permutation
X_lines = @lift(X_coord[to_value($perm)])
Y_lines = @lift(Y_coord[to_value($perm)])

# Endpoints for the new permutation
X_endpoints = @lift(X_coord[to_value($perm)[[1,n]]])
Y_endpoints = @lift(Y_coord[to_value($perm)[[1,n]]])
```
In the above, since `perm` is an Observable, one cannot index a `Vector` by it.
`to_value` is used to access the permutation `perm` contains, which results in a valid reordering operation.
Using these newly defined Observables, we can plot the starting point of the graph.

```{code-block}
lines!(ax, X_lines, Y_lines)
lines!(ax, X_endpoints, Y_endpoints)
fig
```

% In order to make sure `lines!` actually works, we can't separate the code-cells like the above
% so I just execute the code properly in a hidden cell.
```{code-cell}
:tags: ["remove-input"]
fig, ax, splot = scatter(X_coord, Y_coord, axis=(title= @lift("t=$($it)"), ))
lines!(ax, X_lines, Y_lines)
lines!(ax, X_endpoints, Y_endpoints)
fig
```

Now, the basic examples of making animations in `Makie` follow the form
```{code-block} julia
record(fig, "filename.gif", iterator) do it
    ...
end
```
or (an equivalent way of writing the same signature)
```{code-block} julia
record(func, fig, "filename.gif", iterator)
```
where `func` is called for every iteration of `iterator` to create a new frame to be recorded.

It is not clear to me how to use this signature, since we are not going iteration by iteration; after calling `optimize`, `Metaheuristics` does everything for us.
However, reading the documentation for [`record`](https://docs.makie.org/stable/api/#record) reveals an alternative:
```{code-block} julia
record(func, fig, path)
```
where the iteration and frame creation responsibilities are manually handled.
We can combine this with the `logger` mechanism of `optimize`, which allows one to provide a function that will be called with the current `State` after each iteration.
```{code-cell}
:tags: ["remove-output"]
function logger(io, state)
    # get current best solution
    sol = minimizer(state)
    # update the Observables
    perm[] = sol
    it[] = state.iteration
    # record new frame of animation
    recordframe!(io)
end
```

Ultimately, we can obtain the animation with
```{code-block}
record(fig, "tsp.mp4") do io
    optimize(tsp_distance, bounds, ga, 
        logger=(st)->logger(io, st)  # optimize only provides the state, so we need to pass everything properly
    )
end
```

% Unfortunately, I couldn't get the animation to be included automatically.
% So included it manually. Hopefully it won't need to be updated.
<video controls>
    <source src="../_static/tsp.mp4" type="video/mp4">
</video>

## Interactive Example

```{code-cell}
:tags: ["remove-cell"]
best_perms = []
function logger(st)
    push!(best_perms, minimizer(st))
end

ga = GA(;p_mutation = 0.5,
        p_crossover = 0.9,
        initializer = RandomPermutation(N=100), 
        crossover=OrderCrossover(), 
        mutation=SlightMutation(),
        options=options);

result = optimize(tsp_distance, bounds, ga, logger=logger)
```

```{code-cell}
App() do session::Session
    it_slider = Slider(1:result.iteration)
    lab = @lift(string("t=", $(it_slider.value)))
    perm = map((x) -> best_perms[x], it_slider)
    fig, ax, splot = scatter(X_coord, Y_coord, axis=(title=lab, ))
    X_lines = @lift(X_coord[to_value($perm)])
    Y_lines = @lift(Y_coord[to_value($perm)])
    X_endpoints = @lift(X_coord[to_value($perm)[[1,n]]])
    Y_endpoints = @lift(Y_coord[to_value($perm)[[1,n]]])
    lines!(X_lines, Y_lines)
    lines!(X_endpoints, Y_endpoints)
    return Bonito.record_states(session, DOM.div(it_slider, it_slider.value, fig))
end
```