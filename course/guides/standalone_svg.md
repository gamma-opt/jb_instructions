# Producing Standalone SVGs

In this guide, we'll use LaTeX to generate standalone SVGs that can be included in the markdown source files.
There are multiple reasons why this may be desirable.
For example, figures likely don't change as often as the other content in a given page.
Including figures as instructions that get compiled at every build is thus a waste of time, increasingly so as the notes grow larger and there are more figures.
Thus it may make sense to create them separately and include them as pictures at relevant points.

This guide is based on section 10.2.4 of the PGF manual, which can be found [here](https://www.ctan.org/pkg/pgf).

## Steps

1. Create the TeX code for whatever you are interested in, whether is it a Tikz figure or something else.

2. Include it in a barebones LaTeX skeleton for compilation, such as
```latex
\documentclass[dvisvgm]{minimal}
\usepackage{tikz}
\begin{document}
YOUR CODE HERE!!!
\end{document}
```
For convenience, this is provided in the template under `figures/standalone_template.tex`, though you should probably save it under a different name.

3. Compile it to `.dvi` format with `latex file.tex`.

4. Convert the `.dvi` file into `.svg` with `dvisvgm --font-format=woff file.dvi`.
The `WOFF` option should improve the display quality of the images when uploaded online.

```{note}
I expect `dvisvgm` is included in your LaTeX distribution.
If this is not the case and you don't want to obtain it, you can go to an alternative path by changing the document class to `standalone`, compiling to `.pdf` with `pdflatex/xelatex`, and converting to `.svg` with a program like `pdf2svg`.
Though this may itself require installing `pdf2svg`.
```

Possibly useful StackExchange discussion if something goes wrong in the above: [here](https://tex.stackexchange.com/questions/51757/how-can-i-use-tikz-to-make-standalone-svg-graphics).