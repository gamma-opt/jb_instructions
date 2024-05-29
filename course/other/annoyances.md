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