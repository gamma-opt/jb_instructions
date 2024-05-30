# MyST-and-Sphinx Way of Doing Things

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

There are also roles, which are _in-line_ directives, great for things like adding math like {math}`a^2+b^2=c^2` or abbreviations like {abbr}`KKT (Karush–Kuhn–Tucker)` that one can hover over.
The often look like this:
```md
{role-name}`content` 
```
This distinction may be helpful when googling.
````

Everything that is not just regular text is probably some directive.
Some of these features can be achieved with Markdown or LaTeX as well, but using the correct directive often offers more configurability.
If you see something on these pages that you'd like to use, check out its source file.

## Images and Figures

One can add images  the Markdown way (via `![](<link-to-image>))`). Sphinx has an [`{image}` directive](https://docutils.sourceforge.io/docs/ref/rst/directives.html#image), where one can specify layout and alignment.
An often better alternative is the [`{figure}` directive](https://docutils.sourceforge.io/docs/ref/rst/directives.html#figure), which allows for captions and referencing as well.
````md
```{figure} picture.png
:label: example_picture
:align: center

This is the caption.
```
````

```{important}
You can't add a PDF as an image in an HTML file. 
While there seems to be [a mechanism](https://mystmd.org/guide/figures#image-transformers) to automatically convert things when needed, I couldn't get it to work yet.
But a reliable way to make images work, if you'd like to use `.png` in HTML files and `.pdf` in LaTeX, is to save both file types in the same directory, and give the image path as for example `images/myimage.*`.
The appropriate version should be picked automatically for different types of outputs being built.
```

## References

Examples [here](https://jupyterbook.org/en/stable/tutorials/references.html#create-a-citation).

## Footnotes

There is no Sphinx way of adding footnotes.
You can use the Markdown way:
````md
Text with footnote [^1]. More text.

[^1]: I'm the footnote.
````
results in:

Text with footnote [^1]. More text.

[^1]: I'm the footnote.

In HTML outputs, the footnote is put to the bottom of the page.
For more immediate visibility, you may want to use admonition or margin directives instead.

## Further Reading
- [Jupyter Book page](https://jupyterbook.org/en/stable/content/index.html) on various directives and roles.
- MyST pages [here](https://myst-parser.readthedocs.io/en/latest/syntax/roles-and-directives.html) and [here](https://mystmd.org/guide/admonitions).
- Sphinx pages on [directives](https://www.sphinx-doc.org/en/master/usage/restructuredtext/directives.html) and [roles](https://www.sphinx-doc.org/en/master/usage/restructuredtext/roles.html).