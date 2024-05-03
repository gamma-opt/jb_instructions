# Use and Configure

Two very important files in a `jupyter-book` project are `_config.yml` and `_toc.yml`.
The former is used to configure `jupyter-book`, where one can control simple books settings like title and author, but also more complicated ones like notebook caching and Sphinx extensions.

For example, the `_config.yml` of this repo contains the setting that uses the [QuantEcon](https://github.com/QuantEcon/quantecon-book-theme) theme for the generated HTML. 
It also explicitly limits the building to files included in the table of contents, so files don't accidentally get published.

The `_toc.yml` file contains the table of contents that determines the structure of the output.
This website is organized as follows
```yaml
format: jb-book
root: intro
parts:
- caption: Tooling
  chapters:
  - file: tooling/input_and_output
  - file: tooling/config
- caption: Content
  chapters:
  - file: content/markdown_and_sphinx
  - file: content/math_and_code
  - file: content/interactivity
- caption: Other
  chapters:
  - file: other/about
```
which should reflect the table of contents you see on the website, albeit the titles are slightly different.
This is a reasonable, but not the sole way to organize the structure. For more examples, see [here](https://jupyterbook.org/en/stable/structure/toc.html).

```{note}
To add a new page, create a file in the `course` directory and add it to `_toc.yml`.
```

## Further reading
- More on HTML theming [here](https://jupyterbook.org/en/stable/advanced/sphinx.html#choose-a-custom-sphinx-theme) and [here](https://www.sphinx-doc.org/en/master/usage/theming.html).
