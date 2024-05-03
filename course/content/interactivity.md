---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.1
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Interactive Pages

## Letting users execute

One has multiple options to make things interactive when this is deployed as a website.
One idea is to link the pages to interactive computing interfaces like Binder or JupyterHub to allows easy transitioning into a coding environment where people can play around with it themselves.
It is also possible to execute code without leaving the page, using services like Thebe.
For more on both, read [here](https://jupyterbook.org/en/stable/interactive/launchbuttons.html) and [here](https://jupyterbook.org/en/stable/interactive/thebe.html).

## Interactive plots

Another idea that may be nice for an optimization course is interactive plots.
In my googling around, the term "interactive plot" seems to cover a more general concept than what I had in mind.
For example, there are a lot of libraries in various languages that are interactive in the sense that one can filter data and change aesthetic attributes.
Actually changing what is plotted, for example by specifying a new slope for a line or changing the feasible region by adding a constraint, is more difficult because it often requires recomputing some function being plotted.
To do this easily seems to require some amount of JavaScript one way or another.

### In Python

One nice option is [`bokeh`](https://docs.bokeh.org/en/latest/).

Example from [here](https://docs.bokeh.org/en/latest/docs/examples/interaction/js_callbacks/slider.html). On my machine, this was working in Chrome but not in Firefox for some reason.

The `output_notebook()` function is needed for interactivity, but the code block could be hidden in actual lecture notes if needed.

```{code-cell}
import numpy as np

from bokeh.layouts import column, row
from bokeh.models import ColumnDataSource, CustomJS, Slider
from bokeh.plotting import figure, show, output_notebook

output_notebook()
```

```{code-cell}
x = np.linspace(0, 10, 500)
y = np.sin(x)

source = ColumnDataSource(data=dict(x=x, y=y))

plot = figure(y_range=(-10, 10), width=400, height=400)

plot.line('x', 'y', source=source, line_width=3, line_alpha=0.6)

amp = Slider(start=0.1, end=10, value=1, step=.1, title="Amplitude")
freq = Slider(start=0.1, end=10, value=1, step=.1, title="Frequency")
phase = Slider(start=-6.4, end=6.4, value=0, step=.1, title="Phase")
offset = Slider(start=-9, end=9, value=0, step=.1, title="Offset")

callback = CustomJS(args=dict(source=source, amp=amp, freq=freq, phase=phase, offset=offset),
                    code="""
    const A = amp.value
    const k = freq.value
    const phi = phase.value
    const B = offset.value

    const x = source.data.x
    const y = Array.from(x, (x) => B + A*Math.sin(k*x+phi))
    source.data = { x, y }
""")

amp.js_on_change('value', callback)
freq.js_on_change('value', callback)
phase.js_on_change('value', callback)
offset.js_on_change('value', callback)

show(row(plot, column(amp, freq, phase, offset)))
```

### In Julia

I tried getting `PlotlyJS` to work, using an example from [here](https://gist.github.com/sglyon/4e85aa915b30586bddf2c0249e94a58b).

I'm not sure if this is usable with jupyter-book. Currently this is not working with the latest version of JupyterLab, due to an outdated extension (`webio_jupyterlab_extension`). Presumably it would work with an earlier version of JupyterLab (or nbclassic or jupyter notebook), but that doesn't explain why jupyter-book is not working. Somehow, it worked on a ipynb in VSCode though.

There are other Julia packages, as described in `Plots.jl` documentation, but I haven't tried them yet.


### In JavaScript

At one point, it may be the easiest to just do it directly in JavaScript, using probably `Plotly`.

## Further reading
- [Jupyter Book page](https://jupyterbook.org/en/stable/interactive/interactive.html) on interactive plots