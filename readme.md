# Template for course notes/website

## Dependencies
- `jupyter-book`

## Usage

A conda environment to build the website is provided in `dev_env.yml`.

### Build HTML
```
jb build notes
```

### Add a new page
- Create a markdown/Jupyter notebook/MyST markdown file
- Add it to the table of contents (`_toc.yml`)

## Useful References
- [Jupyter Book](https://jupyterbook.org/en/stable/intro.html)
- [quantecon-book-theme](https://github.com/QuantEcon/quantecon-book-theme/blob/master/docs/index.md)

## Todo:
- how to replace QuantEcon logo in theme
  - `btn__qelogo` element