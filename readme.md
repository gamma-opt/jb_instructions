# `jupyter-book` template for course notes/website

## What is this?

This is a [`jupyter-book`](https://jupyterbook.org/en/stable/intro.html)-based template for creating course notes and websites. More specifically, this template aims to:
- Provide a quick overview of `jupyter-book`,
- Show how course content could be created with [Markdown](https://jupyterbook.org/en/stable/file-types/markdown.html) and [MyST Markdown](https://jupyterbook.org/en/stable/file-types/myst-notebooks.html) (or with [notebooks](https://jupyterbook.org/en/stable/file-types/notebooks.html)),
- Present the above in the form of HTML.

There are plenty of related topics and customizations, such as building [a PDF instead of a website](https://jupyterbook.org/en/stable/advanced/pdf.html) and [Sphinx customization](https://jupyterbook.org/en/stable/sphinx/index.html) that should be linked, but may not be explictly discussed.

To go through this template, you can either browse the files in the `course` directory manually, or build the website following the instructions below.

## Usage

### (Optional) Create an environment

A `conda` environment to build the website is provided in `environment.yml`. If you don't have `conda`, you can get it from [here](https://docs.conda.io/en/latest/). You can also try [`mamba` or `micromamba`](https://mamba.readthedocs.io/en/latest/index.html) for a faster alternative.

Create the environment with
```
conda env create -f environment.yml
```
and activate via
```
conda activate jb_course
```
 If you are about to develop a new course, you may want to give a more specific name to the environment using `-n`.

### Build the HTML files

Running
```
jb build course
```
will build the HTML files, which you can find in `course/_build/html`. Open `index.html` to start browsing.


You may also want to use
```
jb clean course
```
to clean the `course/_build` directory occasionally.

## Useful References
- [Jupyter Book](https://jupyterbook.org/en/stable/intro.html)
- [MyST Markdown](https://myst-parser.readthedocs.io/en/latest/index.html)
