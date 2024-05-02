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
  display_name: Julia 1.10.2
  language: julia
  name: julia-1.10
---

# Example code with synced tabs

```{contents} Contents
:depth: 2
```

## Problem 1

Here we create a function that takes in a list of numbers and outputs another list with every element of the first squared.

### Implementation Example

% This is maybe a little too verbose, but shorter solutions capitalize tab-labels, which look ugly
`````{tab-set}

````{tab-item} Julia
:sync: julia_tab
```julia
function square_list(l)
    n .^ 2
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

## Problem 2

Here we create a function that takes in a number `x` and outputs `x+1`.

### Implementation Example

`````{tab-set}

````{tab-item} Julia
:sync: julia_tab
```julia
function square_list(x)
    x+1
end
```
````

````{tab-item} Python
:sync: python_tab
```python
def square_list(x):
    return x+1
```
````

`````