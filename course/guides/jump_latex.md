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

# Manipulating LaTeX Outputs of JuMP

Suppose you printed an objective function in JuMP, but it is very long and doesn't fit into the page (either in HTML or LaTeX).

```{code-cell}
using JuMP

model = Model()
@variable(model, x[1:30] >= 0)
@objective(model, Min, sum(x))
```

or 

```{code-cell}
print(model)
```

In these cases, it may be worth considering just using a summary like `show(model)`.
However, if printing is necessary, we can still do so, by modifying the underlying LaTeX string.

For the objective function, we can access the string with
```{code-cell}
s = objective_function_string(MIME("text/latex"), model)
```
This gives a string that we can modify, for example by adding a line break with `\\\\` (special characters need escaping).
```{code-cell}
s = s[begin:123] * "\\\\" * s[124:end]
```

Once we have what we'd like printed, we can render it using the display function.
One thing to take note is that the string still requires being put in an appropriate environment.
```{code-cell}
display("text/latex","\\begin{align*}"*s*"\\end{align*}")
```

For printing the entire model, the same logic applies, but accessing the string is slightly different.
One way of doing so is by using buffers
```{code-cell}
io = IOBuffer()
JuMP._print_latex(io, model)
s = String(take!(io))
```
This one is likely much longer and it comes pre-wrapped in a math environment, so one only needs to apply the desired modifications.

When including such a content, you can make use of cell tags to hide the modification code.
For example with
````md
```{code-cell}
:tags: [remove-output]

objective_function(model)
```

```{code-cell}
:tags: [remove-input]

s = objective_function_string(MIME("text/latex"), model)
s = s[begin:123] * "\\\\" * s[124:end]
display("text/latex","\\begin{align*}"*s*"\\end{align*}")
```
````
the resulting HTML/LaTeX will look as if the output was uninterrupted.
```{code-cell}
:tags: [remove-output]

objective_function(model)
```

```{code-cell}
:tags: [remove-input]

s = objective_function_string(MIME("text/latex"), model)
s = s[begin:123] * "\\\\" * s[124:end]
display("text/latex","\\begin{align*}"*s*"\\end{align*}")
```