# MyST and Sphinx

There are a lot of features that may come up when designing notes/webpages.
If you've followed this guide linearly so far, you should have seen notes on margins, highlighted text (also called admonitions) and will see synced tabs, code output and possibly other things too.
These are often implemented via directives, which are blocks of explicit markup that often look like this:
````md
```{directive-name}
{additional-config}
content
```
````

````{dropdown} I am a directive too! Click me!

There are also roles, which are _in-line_ directives, great for things like adding math like {math}`a^2+b^2=c^2` or abbreviations like {abbr}`KKT (Karush–Kuhn–Tucker)`.
The often look like this:
```md
{role-name}`content` 
```
This distinction may be helpful when googling.
````

Everything that is not just regular text is probably some directive.
Some of these features can be achieved with Markdown or LaTeX as well, but using the correct directive often offers more configurability.
If you see something on these pages that you'd like to use, check out its source file.

## Further Reading
- [Jupyter Book page](https://jupyterbook.org/en/stable/content/index.html) on various directives and roles.
- MyST pages [here](https://myst-parser.readthedocs.io/en/latest/syntax/roles-and-directives.html) and [here](https://mystmd.org/guide/admonitions).
- Sphinx pages on [directives](https://www.sphinx-doc.org/en/master/usage/restructuredtext/directives.html) and [roles](https://www.sphinx-doc.org/en/master/usage/restructuredtext/roles.html).