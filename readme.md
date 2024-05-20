[![GitHub Pages with jupyter-book](../../actions/workflows/jupyterbook-ghpages.yml/badge.svg)](../../actions/workflows/jupyterbook-ghpages.yml)
[![jupyterbook-pdf](../../actions/workflows/jupyterbook-pdf.yml/badge.svg)](../../actions/workflows/jupyterbook-pdf.yml)
[![](https://img.shields.io/badge/View%20on%20GitHub%20Pages-blue)](https://gamma-opt.github.io/jb_instructions/intro.html)

# Instructions for our `jupyter-book` template (for course notes/website)

## What is this?

This repo contains instructions on using the [`jupyter-book`](https://jupyterbook.org/en/stable/intro.html)-based templatewe have [here](https://github.com/gamma-opt/jb_course_template) for creating course notes and websites. 
This instructions repository itself is built on top of the template!

You can browse this repository best by looking at its [Github Pages deployment](https://gamma-opt.github.io/jb_instructions/intro.html), or build it in either HTML or PDF form following the instructions below.

## Building

### (Optional) Create an environment

A `conda` environment to build the website is provided in `environment.yml`. If you don't have `conda`, you can get it from [here](https://docs.conda.io/en/latest/). You can also try [`mamba` or `micromamba`](https://mamba.readthedocs.io/en/latest/index.html) for a faster alternative.

Create the environment with
```bash
conda env create -f environment.yml
```
and activate via
```bash
conda activate jb_course
```
You can create the `conda` environment with a different name using `-n`.

Also make sure Julia is ready with
```julia
using Pkg
Pkg.activate(".")
Pkg.instantiate()
 ```

### Build the HTML files

Running
```
jb build course
```
will build the HTML files, which you can find in `course/_build/html`. Open `index.html` to start browsing.

### Built the PDF

For this, you need LaTeX.
If you don't have LaTeX and don't know how to install it, I recommend building the HTML.

```bash
jb build course --builder pdflatex
```

### Cleanup

You may also want to use
```
jb clean course
```
to clean the `course/_build` directory occasionally.
