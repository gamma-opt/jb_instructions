# Annoyances

I'll keep track of "big" problems or unintuitive solutions to them here.

(annoyance:multiple-labels)=
## Having multiple labels in equations

[Relevant GitHub issue](https://github.com/sphinx-doc/sphinx/issues/12379)

Suppose you'd like to type the following equation:
```{image} ../_static/multiple_labels.png
:name: Equations with multiple labels
```

It turns out that the `{math}` directive doesn't support multiple labels.
One can do
````md
```{math}
:label: multiple-labels-1
& Ax = b, \ \ x \ge 0,

& A^\top p + u = c, \ \ u \ge 0,

& u^\top x = 0.
```
````
to get
```{math}
:label: annoyance:multiple-labels-1
& Ax = b, \ \ x \ge 0,

& A^\top p + u = c, \ \ u \ge 0,

& u^\top x = 0.
```
however we end up with only a single label we can refer to with ``{eq}`annoyance:multiple-labels-1` ``: {eq}`annoyance:multiple-labels-1`.

We can try using LaTeX commands
````md
\begin{align*}
& Ax = b, \ \ x \ge 0, \label{annoyance:eq1}\tag{1} \\
& A^\top p + u = c, \ \ u \ge 0, \label{annoyance:eq2}\tag{2} \\
& u^\top x = 0. \label{annoyance:eq3}\tag{3}
\end{align*}
````
to get
\begin{align*}
& Ax = b, \ \ x \ge 0, \label{annoyance:eq1}\tag{1} \\
& A^\top p + u = c, \ \ u \ge 0, \label{annoyance:eq2}\tag{2} \\
& u^\top x = 0. \label{annoyance:eq3}\tag{3}
\end{align*}
but referring to them requires LaTeX as well:
- ``{eq}`annoyance:eq2` `` doesn't work (you can try it for yourself, I removed the example to avoid getting a warning)
- ``{math}`\eqref{annoyance:eq2}`` does: {math}`\eqref{annoyance:eq2}`

The obvious problem is that Sphinx references and LaTeX references are tracked separately.
One less obvious problem is that Sphinx references work in between different pages, whereas LaTeX references (at least in the case of HTML) don't seem to work.

One can manually keep track and make sure all the `\tag` commands are correct, but that is annoying, especially if you'd like to add a new equation earlier than previously written equations.

(annoyance:mermaid-math)=
## Broken math and PDFs for Mermaid.js diagrams

Currently Mermaid diagrams in this website doesn't render math correctly.
I'm pretty sure the reason is some bug in `sphinxcontrib-mermaid` but I haven't investigated it yet.
A way to circumvent this is to set `mermaid_output_format` in `_config.yml` to `png` or `svg` so that instead of the diagram being rendered in HTML, it is built as an image, then attached.
This is how it gets added to PDF files as well.
This process requires installing the `@mermaid-js/mermaid-cli` Node package, and will normally call `mmdc` provided by it.

Here, there is a decision to make: you can install `@mermaid-js/mermaid-cli` globally (e.g. with `npm install -g @mermaid-js/mermaid-cli`), which will make `mmdc` available in `$PATH`.
However, I don't know of a clean way of specifying a global install in the dependencies of the template.
The alternative is to install it normally, then set the `mermaid_cmd` in `_config.yml` to be something like `npx mmdc`.
This would work, except it doesn't due to a bug in `sphinxcontrib-mermaid` (Github issue [here](https://github.com/mgaitan/sphinxcontrib-mermaid/issues/89)).
My hope is that I can get more activity in the issue (and make a PR myself if needed).
If not, an alternative solution is to change the package management in a way that would make keeping track of things easier, for example using a Nix flake.

(annoyance:qbt-version)=
## `quantecon-book-theme`-imposed version constraints

Creating a `conda` environment with the current `environment.yml` will result in having `sphinx==5.3` (even though `conda` may report is as a more recent version).
This is because pip installing `quantecon-book-theme` downgrades `sphinx`.
This is now annoying because some configuration options for PDF (like `sphinxlightbox`-based admonitions, e.g. `{note}`) are added in a later version of `sphinx`.
Naively removing the constraints in `quantecon-book-theme` broke the sidebar table of contents.
So for now, this is left in an unideal situation.
I made an [issue](https://github.com/QuantEcon/quantecon-book-theme/issues/247) about updating the theme, but if they don't I may need to do it myself.

(annoyance:svg-issues)=
## SVG-related problems

This is relatively minor, but I'm writing down for the sake of having a record.
First of all, and this is not (yet) a problem, the full potential of SVGs is not yet available in `sphinx` (and thus `jupyter-book`).
By that I mean that directives like 
````md
```{figure} file.svg
```
````
get converted to `<img>` tag in HTML, instead of `<svg>` or `<object>`.
Related issue [here](https://github.com/sphinx-doc/sphinx/issues/2240)

Also, using `:scale:` with SVGs is broken in `sphinx` for multiple reasons.
One problem is that SVGs being XML documents, they accept single- or double-quoted strings, so `width="10pt"` and `width='10pt'` are equally valid.
Some programs, like `dvisvgm` output single-quoted strings (as visible [here](https://github.com/mgieseki/dvisvgm/blob/1790f69b363945b1be30d463a93b2b45ab71195f/src/XMLNode.cpp#L384)).
`sphinx` then uses a Python package called [`imagesize`](https://github.com/shibukawa/imagesize_py) to obtain the dimensions in the SVG, so that it can apply scaling.
However, this is done with regex matching [here](https://github.com/shibukawa/imagesize_py/blob/8d88ec6b646d6184b5633604551d6fc154783073/imagesize/imagesize.py#L210-L211) that is hard-coded to double-quotes.
Thus in single-quoted SVGs, the sizes are not obtained, so `sphinx` doesn't apply scaling.
One can get around this by using a different library for obtaining SVGs, and `pdftocairo` seems good to me.
Related issues in [`sphinx`](https://github.com/sphinx-doc/sphinx/issues/12415) and [`imagesize`](https://github.com/shibukawa/imagesize_py/issues/64).

However, even if you have a double-quoted SVG, if the length units are in `pt`, conversion to `px` happens incorrectly, which leads to incorrect scaling.
Related issue [here](https://github.com/shibukawa/imagesize_py/issues/42).

Using `:height:` or `:width:` seem to work fine, and should circumvent the above issues.