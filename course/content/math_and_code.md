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
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Math, Code Formatting and Execution

## Math

There are multiple ways one can include mathematical expressions in MyST-Markdown.

### Sphinx role/directive

One is using the `math` role or directive from sphinx to create inline and block math:
````md
Let {math}`f(x_1,x_2)` represent our objective function.
```{math}
:label: problem_1
\min_{x_1,x_2} \quad& f(x_1,x_2) \\
\text{s.t.} \quad& x_1\geq 0 \\
& x_2 \geq 0 \\
& x_1+x_2 \leq 0
```
The label option lets you refer to {eq}`problem_1`.
````
will result in:


Let {math}`f(x_1,x_2)` represent our objective function.
```{math}
:label: problem_1
\min_{x_1,x_2} \quad& f(x_1,x_2) \\
\text{s.t.} \quad& x_1\geq 0 \\
& x_2 \geq 0 \\
& x_1+x_2 \leq 0
```
The label option lets you refer to {eq}`problem_1`.

### LaTeX-style math

Having enabled `amsmath` in `_config.yml`, one can also use the math environments
> equation, multline, gather, align, alignat, flalign, matrix, pmatrix, bmatrix, Bmatrix, vmatrix, Vmatrix, eqnarray

and the unnumbered equivalents with `*`.

```{warning}
`\label`s inside these environments may not work properly, so if you'd like to refer to equations, prefer the directive described above. For multiple labels in the same environment, make sure you read {ref}`annoyance:multiple-labels`.
```

```{attention}
Avoid using `$$` for math environments, in my random testing it didn't work well sometimes.
```

### Mathematical Statements

For formatting mathematical text like definitions and proofs, the [`sphinx-proof`](https://sphinx-proof.readthedocs.io/en/latest/index.html) package is used.
This package defines a number of additional directives, such as `{prf:proof}` and `{prf:algorithm}`.
Check the website for a complete list.

````md
```{prf:algorithm} Barrier method for LP
:label: p1c7:alg:barrier_method

1. **initialise.** (primal-dual feasible) $(x^0, p^0, u^0)$, $\epsilon > 0$, $\mu_0 = \mu_1>0$,

2. $\beta \in (0,1)$, $k = 0$. 

3. **while** $n \mu_k > \epsilon$

    1. compute $d^{k+1} = (d_x^{k+1}, d_p^{k+1}, d_u^{k+1})$

    2. compute appropriate step size $\theta^{k+1} = (\theta_p^{k+1}, \theta_d^{k+1}$)

    3. $(x^{k+1}, p^{k+1}, u^{k+1}) = (x^k, p^k, u^k) + \theta^{k+1}d^{k+1}$

    4. $k = k+1$

    5. $\mu_{k+1} = \beta\mu_k$

4. **end while**

5. **return** $(x^k, p^k, u^k)$.
```
````

```{prf:algorithm} Barrier method for LP
:label: p1c7:alg:barrier_method

1. **initialise.** (primal-dual feasible) $(x^0, p^0, u^0)$, $\epsilon > 0$, $\mu_0 = \mu_1>0$,

2. $\beta \in (0,1)$, $k = 0$. 

3. **while** $n \mu_k > \epsilon$

    1. compute $d^{k+1} = (d_x^{k+1}, d_p^{k+1}, d_u^{k+1})$ using {math}`\eqref{p1c7:eq:infeasible_perturbed_system}`

    2. compute appropriate step size $\theta^{k+1} = (\theta_p^{k+1}, \theta_d^{k+1}$)

    3. $(x^{k+1}, p^{k+1}, u^{k+1}) = (x^k, p^k, u^k) + \theta^{k+1}d^{k+1}$

    4. $k = k+1$

    5. $\mu_{k+1} = \beta\mu_k$

4. **end while**

5. **return** $(x^k, p^k, u^k)$.
```

When using this package and compiling a PDF, `sphinx-proof` seems to automatically add a proof index at the end.
I disabled it in the template by adding to the config `latex_domain_indices: false `, but it can be enabled back if desired.

## Code

### Formatted Code

Syntax highlighting is simple:

````md
```python
import numpy as np
a = np.random.random()
```
````
gives
```python
import numpy as np
a = np.random.random()
```

You can even have nice synced tabs like this:

`````{tab-set}
````{tab-item} Julia
:sync: julia_tab
```julia
function square_list(l)
    l .^ 2
end
```
````

````{tab-item} Python
:sync: python_tab
```python
def square_list(l):
    return [x**2 for x in l]
```
````
`````

`````{tab-set}
````{tab-item} Julia
:sync: julia_tab
```julia
function add_one(x)
    x+1
end
```
````

````{tab-item} Python
:sync: python_tab
```python
def add_one(x):
    return x+1
```
````
`````

Look at the Markdown file of this page to see how.

### Executable Code

````{attention}
If you want to execute code, you need to add a MyST-Markdown top-matter to the source file, or use a `.ipynb` file. 
You can do so easily with
```bash
jupyter-book myst init mymarkdownfile.md --kernel kernelname
```
If you don't know the kernel name, you can list them with
```bash
jupyter kernelspec list
```
````

Code to be executed should be included in `{code-cell}`s.
````md
```{code-cell}
1+1
```
````
gives
```{code-cell}
1+1
```

```{important}
While we are working with Markdown files, at the end of the day these are executed while being interpreted as notebooks. Thus, you can only have one kernel associated with a file, which is defined in the top-matter. One cannot execute Python code in one block and Julia in another in the same file.
```

There are also lots of options to work with `code-cell`s, such as hiding the input, making cells scrollable, and formatting the outputs. You can read more [here](https://jupyterbook.org/en/stable/content/executable/index.html).

```{note}
One thing I encountered while preparing this was that when working with Julia, if you create some files but then update Julia version, one may need to rebuild `IJulia` to get things working again.
```

## Further reading
- Math pages from [Jupyter Book](https://jupyterbook.org/en/stable/content/math.html), [MyST-Markdown](https://myst-parser.readthedocs.io/en/latest/syntax/math.html), and [Sphinx](https://www.sphinx-doc.org/en/master/usage/restructuredtext/directives.html#math).
- [More](https://myst-parser.readthedocs.io/en/latest/syntax/code_and_apis.html) on syntax highlighting, including numbering and highlighting lines, including code from files and inline highlighting.
