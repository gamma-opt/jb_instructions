# Source files and outputs

The basic idea of this template is to organize the content of some course using Markdown files and presented it as a reasonably nice looking website.
These webpages were all built from Markdown files, some of which are regular and others containing executed code and interactive plots.

## Source files

I assume you have a general idea of what Markdown is and what you can do with it, if not [this](https://www.markdownguide.org/basic-syntax) can give you some idea.

Markdown is nice and its basic features are often enough to organize information such as course notes or websites. 
Most notably however, executing code and displaying its output is not one of these features. 
To solve this problem, often people use something like Jupyter Notebooks or Quarto to have code alongside Markdown.
In `jupyter-book` projects, MyST-Markdown (standing for Markedly Structured Text) is used, which extends Markdown with
- the ability to encode notebooks as Markdown files (using [`MyST-NB`](https://myst-nb.readthedocs.io/en/latest/index.html)), so we get the niceties of working with code without the annoyance of the notebook file changing its state everytime you open it/execute some cell (preserving the sanity of `git status`), and
```{margin}
Sphinx is a powerful documentation generator.
```
- Sphinx compatibility, which allows for a high level of customizability in the output.

To embed a notebook in a MyST-Markdown file, one needs a YAML top-matter.
To create it easily, one can create the Markdown file themselves, and ask `jupyter-book` to genereate the top-matter:
```bash
jupyter-book myst init mymarkdownfile.md --kernel kernelname
```

```{important}
Every file must have a title, which usually means it should begin with a top-level header `#`, and only one top-level header should be present. Also, don't skip levels, i.e. don't jump from `#` to `###`.
```

## Outputs

This template is primarily geared towards producing HTML files to be served as a static website.
`jupyter-book` also allows one to build either a single PDF containing everything (via LaTeX or HTML using `pyppeteer`) or individual PDF files for each Markdown file (LaTeX only). You can read more about this and more [here](https://jupyterbook.org/en/stable/basics/build.html).

## Further reading
- Jupyter Book pages on [source files](https://jupyterbook.org/en/stable/file-types/index.html) and [oublishing](https://jupyterbook.org/en/stable/basics/building/index.html)
- [MyST Markdown](https://myst-parser.readthedocs.io/en/latest/index.html)
- [Sphinx](https://www.sphinx-doc.org/en/master/) documentation for advanced use.
