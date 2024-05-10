# Notes on design

There are two issues to consider here that work separately: the design of the website and the design of the PDF produced (assuming the latter happens via LaTeX).

Currently, the PDF design follows the default options.

The website uses [QuantEcon](https://github.com/QuantEcon/quantecon-book-theme) theme for the generated HTML. But there are a couple things to note:
- The logo on the right margin (page-toc) is easily changeable in `_config.yml`.
- The logo on the toolbar was originally a logo of QuantEcon, hardcoded into the CSS files provided by the theme. You can change it by providing custom CSS with the `!important` keyword, an example is included in `_static/style.css`.
- There is a button included change the contrast/make it dark mode, but it seems to not work with admonitions (see [](../content/math_and_code.md)).
- To make the Github button work, make sure you don't rename the folder if you clone the repo.
  - Why? Because [this function](https://github.com/QuantEcon/quantecon-book-theme/blob/555f1c8897a9f6a40c2f653fc2bfcf84be30f040/src/quantecon_book_theme/__init__.py#L163-L173) is checking for that, and gets used [here](https://github.com/QuantEcon/quantecon-book-theme/blob/555f1c8897a9f6a40c2f653fc2bfcf84be30f040/src/quantecon_book_theme/theme/quantecon_book_theme/layout.html#L253) to create the hyperlink for the Github button.


## List of not-obvious modifications
- `_static/style.css` replaces the toolbar logo with `_static/logo.png`
- `_static/remove_pdf_button.js` removes the "Download PDF" button from pages.