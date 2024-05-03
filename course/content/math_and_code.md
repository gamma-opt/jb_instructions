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
`\label`s inside these environments may not work properly, so if you'd like to refer to equations, prefer the directive described above.
```

### More

For formatting mathematical text like definitions and proofs, see [here](https://jupyterbook.org/en/stable/content/proof.html).

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

```{attention}
Don't forget to make sure the source file contains the MyST-Markdown top-matter if you want to execute code.
```

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
