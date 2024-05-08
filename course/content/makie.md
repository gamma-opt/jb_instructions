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


Formatting in this page is a bit broken because the last example is using `TailwindDashboard`, which overwrites some styling.
I'll remove it, add other examples and annotate what is going on/make this a better form for the template when a bug fix is released ([status here](https://github.com/JuliaRegistries/General/pull/106399)).


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
app = App() do
    return DOM.div(DOM.h1("hello world"), js"""console.log('hello world')""")
end
```

```{code-cell}
App() do session
    s = Slider(1:3)
    value = map(s.value) do x
        return x ^ 2
    end
    # Record states is an experimental feature to record all states generated in the Julia session and allow the slider to stay interactive in the statically hosted docs!
    return Bonito.record_states(session, DOM.div(s, value))
end
```

```{code-cell}
import Bonito.TailwindDashboard as D
app = App() do session
    colors = ["black", "gray", "silver", "maroon", "red", "olive", "yellow", "green", "lime", "teal", "aqua", "navy", "blue", "purple", "fuchsia"]
    nsamples = D.Slider("nsamples", 1:200, value=100)
    nsamples.widget[] = 100
    sample_step = D.Slider("sample step", 0.01:0.01:1.0, value=0.1)
    sample_step.widget[] = 0.1
    phase = D.Slider("phase", 0.0:0.1:6.0, value=0.0)
    radii = D.Slider("radii", 0.1:0.1:60, value=10.0)
    radii.widget[] = 10
    svg = DOM.div()
    evaljs(session, js"""
        const [width, height] = [900, 300]
        const colors = $(colors)
        const observables = $([nsamples.value, sample_step.value, phase.value, radii.value])
        function update_svg(args) {
            const [nsamples, sample_step, phase, radii] = args;
            const svg = (tag, attr) => {
                const el = document.createElementNS('http://www.w3.org/2000/svg', tag);
                for (const key in attr) {
                    el.setAttributeNS(null, key, attr[key]);
                }
                return el
            }
            const color = (i) => colors[i % colors.length]
            const svg_node = svg('svg', {width: width, height: height});
            for (let i=0; i<nsamples; i++) {
                const cxs_unscaled = (i + 1) * sample_step + phase;
                const cys = Math.sin(cxs_unscaled) * (height / 3.0) + (height / 2.0);
                const cxs = cxs_unscaled * width / (4 * Math.PI);
                const circle = svg('circle', {cx: cxs, cy: cys, r: radii, fill: color(i)});
                svg_node.appendChild(circle);
            }
            $(svg).replaceChildren(svg_node);
        }
        Bonito.onany(observables, update_svg)
        update_svg(observables.map(x=> x.value))
        """)
    return DOM.div(D.FlexRow(D.FlexCol(nsamples, sample_step, phase, radii), svg))
end
```


```{code-cell}
app = App() do session
    text = TextField("test")
    slider = Slider(1:10, startvalue = 2)
    slider_val = DOM.p(slider[])
    fig, ax, plot = ablines(0,2)

    onjs(session, text.value, js"(v) => console.log(v)")
    #onjs(session, slider.value, js"""function on_update(new_val) {
    #    const p_element = $(slider_val)
    #    p_element.innerText = new_val
    #    $(plot).then(plots=>{
    #        
    #    })
    #}
    #""")

    Bonito.on_document_load(session, js"""
        $(plot).then(plots=>{
            const abline = plots[0]
            console.log(abline)
        })
    """)

    return DOM.div(text, slider, slider_val, fig)
end
```


```{code-block}
App() do session::Session
    s1 = Slider(1:100)
    slider_val = DOM.p(s1[])
    fig, ax, splot = scatter(1:4)

    on_document_load(session, js"""
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