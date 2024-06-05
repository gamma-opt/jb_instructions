# Building a PDF

One can use
```bash
jb build course --builder pdflatex
```
to create a PDF instead of HTML output.

With the default options, the PDF output is not perfect.
For example, margins and code-blocks/cells have a lot of space for improvement, and admonitions could use some coloring.
The template contains some changes to the default options, most if not all of which can be found under `sphinx:config:latex_elements`.
The documentation for this configuration can be found [here](https://www.sphinx-doc.org/en/master/latex.html).

## Notes
- `sphinx-proof` at least for some of its directives uses the `{note}` directive.
This means that changing the styling of `{note}` will also affect them, for example `{algorithm}`.
- `quantecon-book-theme`-related constraints lead to having an old version of `sphinx`, which doesn't have every `sphinxsetup` key used in the template. 
For more details, see {ref}`annoyance:qbt-version`.
- `fontspec` is used to set DejaVu Sans Mono (the font used in Julia documentation) as the font for code-blocks (via `fancyvrb`).
`fontspec` doesn't work with `pdflatex`, but `jupyter-book` by default uses `xelatex` anyway so this shouldn't be a problem.
- Source Sans Pro is set as the default text font, matching the HTML theme.
Size is set to 11pt, because I thought 10 was too small and 12 too big.
- Titles, links and admonitions are colored using the (shades of the) first four colors [here](https://coolors.co/0072bc-ffffff-c46e71-40c9a2-522b47).
