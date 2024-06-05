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
---

# Admonitions Gallery

```{tip}
A tip
```

```{attention}
Attention!
```

```{caution}
There are traps everywhere!
```

```{danger}
It's actually pronounced Donger. It's Dutch.
```

```{error}
404
```

```{hint}
Mint
```

```{important}
!!!!!!!!!!!!!!
```

```{note}
Remember to fix everything
```

```{seealso}
aaaaaa
```

```{warning}
bbbbbbbbbbbbb
```

```{admonition} I have a title
:class: tip

I have content
```

````{raw} latex
\begin{sphinxVerbatim}[commandchars=\\\{\}]
function mandelbrot(a)
\end{sphinxVerbatim}
````

````{raw} latex
\begin{Verbatim}
function mandelbrot(a)
\end{Verbatim}
````

```julia
function square_list(l)
    l .^ 2
end
```

```{code-cell}
function square_list(l)
    l .^ 2
end

square_list([1,2,3])
```

The quick brown fox jumps over the sleazy dog.