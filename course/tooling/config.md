# Use and Configure

Two very important files in a `jupyter-book` project are `_config.yml` and `_toc.yml`.
The former is used to configure `jupyter-book`, where one can control simple books settings like title and author, but also more complicated ones like notebook caching and Sphinx extensions.

For example, the `_config.yml` of the template contains the setting that uses the [QuantEcon](https://github.com/QuantEcon/quantecon-book-theme) theme for the generated HTML. 
It also explicitly limits the building to files included in the table of contents, so files don't accidentally get published.

The `_toc.yml` file contains the table of contents that determines the structure of the output.
This website is organized as follows
```yaml
format: jb-book
root: intro
parts:
- caption: Guides
  chapters:
  - file: guides/new_course
  - file: guides/update_from_template
- caption: Tooling
  chapters:
  - file: tooling/input_and_output
  - file: tooling/config
...
```
which should reflect the table of contents you see on the website, albeit the titles may be slightly different.
This is a reasonable, but not the sole way to organize the structure. For more examples, see [here](https://jupyterbook.org/en/stable/structure/toc.html).

```{note}
To add a new page, create a file somewhere in the `course` directory and add it to `_toc.yml`.
```

## Numbering Chapters

By default, chapters are unnumbered.
Enabling numbering can depend on the structure of your `_toc.yml` and how you want to configure it, but for the above example, it is done by adding `numbering: true` under every caption.


## Notes on design

There are two issues to consider here that work separately: the design of the website and the design of the PDF produced (assuming the latter happens via LaTeX).

Currently, the PDF design follows the default options.

The website uses [QuantEcon](https://github.com/QuantEcon/quantecon-book-theme) theme for the generated HTML. But there are a couple things to note:
- The logo on the right margin (page-toc) is easily changeable in `_config.yml`.
- The logo on the toolbar was originally a logo of QuantEcon, hardcoded into the CSS files provided by the theme. You can change it by providing custom CSS with the `!important` keyword, an example is included in `_static/style.css`.
- There is a button included change the contrast/make it dark mode, but it seems to not work with admonitions (see [](../content/math_and_code.md)).
- To make the Github button work, make sure you don't rename the folder if you clone the repo.
  - Why? Because [this function](https://github.com/QuantEcon/quantecon-book-theme/blob/555f1c8897a9f6a40c2f653fc2bfcf84be30f040/src/quantecon_book_theme/__init__.py#L163-L173) is checking for that, and gets used [here](https://github.com/QuantEcon/quantecon-book-theme/blob/555f1c8897a9f6a40c2f653fc2bfcf84be30f040/src/quantecon_book_theme/theme/quantecon_book_theme/layout.html#L253) to create the hyperlink for the Github button.


### List of not-obvious modifications
- `_static/style.css` replaces the toolbar logo with `_static/logo.png`
- `_static/remove_pdf_button.js` removes the "Download PDF" button from pages.

## Further reading
- More on HTML theming [here](https://jupyterbook.org/en/stable/advanced/sphinx.html#choose-a-custom-sphinx-theme) and [here](https://www.sphinx-doc.org/en/master/usage/theming.html).
- [Jupyter Book configuration reference](https://jupyterbook.org/en/stable/customize/config.html)