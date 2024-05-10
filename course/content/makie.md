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
  display_name: Julia 1.10.3
  language: julia
  name: julia-1.10
---

# Interactive plots in Julia

As a simple example, one can rotate and zoom in/out with 3D plots.

```{code-cell}
using WGLMakie, Bonito
Page(exportable=true, offline=true)
WGLMakie.activate!()

N = 60
function xy_data(x, y)
    r = sqrt(x^2 + y^2)
    r == 0.0 ? 1f0 : (sin(r)/r)
end
l = range(-10, stop = 10, length = N)
z = Float32[xy_data(x, y) for x in l, y in l]
surface(
    -1..1, -1..1, z,
    colormap = :Spectral
)
```

```{code-cell}
App() do session
    s = Slider(1:5, startvalue=2)
    value = map(s.value) do x
        return x ^ 2
    end
    # Record states is an experimental feature to record all states generated in the Julia session and allow the slider to stay interactive in the statically hosted docs!
    return Bonito.record_states(session, DOM.div(s, value))
end
```

```{code-cell}
app = App() do session
    slider = Slider(1:10)
    slider[] = 2  # startvalue doesn't work, do it manually
    slider_val = DOM.p(slider[])

    onjs(session, slider.value, js"""function on_update(new_val) {
        const p_element = $(slider_val)
        p_element.innerText = new_val
    }
    
    """)

    Bonito.on_document_load(session, js"""
        const p_element = $(slider_val)
        console.log("slider: ", p_element.innerText)
    """)

    return DOM.div(slider, slider_val)
end
```
These sliders can be tied to various aspects of the plot (found under `geometry` or `material`). 
In the below graph, the slider value is used as the slope.
The Javascript code calculates a new end-point for the line segment and updates it.
There is also a text box but it is not doing anything yet.

```{code-cell}
app = App() do session
    text = TextField("test")
    slider = Slider(1:10, startvalue = 2)
    slider_val = DOM.p(slider[])
    fig, ax, plot = ablines(0,2)

    onjs(session, text.value, js"(v) => console.log(v)")
    onjs(session, slider.value, js"""function on_update(new_val) {
        $(plot).then(plots=>{
            const abline = plots[0]
            const x = abline.geometry.attributes.linepoint_end.data.array[4]
            abline.geometry.attributes.linepoint_end.data.array[5] = x*new_val
            abline.geometry.attributes.linepoint_end.needsUpdate = true
        })
    }
    """)
    onjs(session, slider.value, js"""function on_update(new_val) {
        const p_element = $(slider_val)
        p_element.innerText = new_val
    }
    
    """)

    Bonito.on_document_load(session, js"""
        $(plot).then(plots=>{
            const abline = plots[0]
            console.log(abline)
        })

        const p_element = $(slider_val)
        console.log("slider: ", p_element.innerText)
    """)

    return DOM.div(text, slider, slider_val, fig)
end
```
Here the slider moves the first point.

```{code-cell}
app = App() do session::Session
    s1 = Slider(1:100)
    slider_val = DOM.p(s1[])
    fig, ax, splot = scatter(1:4)

    Bonito.on_document_load(session, js"""
        $(splot).then(plots=>{
            const scatter_plot = plots[0]
            console.log(scatter_plot)
        })
    """)

    onjs(session, s1.value, js"""function on_update(new_value) {
        $(splot).then(plots=>{
            const scatter_plot = plots[0]
            scatter_plot.geometry.attributes.pos.array[0] = (new_value/100) * 4
            scatter_plot.geometry.attributes.pos.array[1] = (new_value/100) * 4
            scatter_plot.geometry.attributes.pos.needsUpdate = true
        })
    }
    """)
    return DOM.div(s1, fig)
end
```

## Further reading
- [WGLMakie docs](https://docs.makie.org/stable/explanations/backends/wglmakie/)