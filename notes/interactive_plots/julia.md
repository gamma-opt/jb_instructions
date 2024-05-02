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
  display_name: Julia 1.10.2
  language: julia
  name: julia-1.10
---

# Interactive plots in Julia

Interactive plots in Python using `PlotlyJS`. Example from [here](https://gist.github.com/sglyon/4e85aa915b30586bddf2c0249e94a58b).

I'm not sure if this is usable with jupyter-book. Currently this is not working with the latest version of JupyterLab, due to an outdated extension (`webio_jupyterlab_extension`). Presumably it would work with an earlier version of JupyterLab (or nbclassic or jupyter notebook), but that doesn't explain why jupyter-book is not working. Somehow, it worked on a ipynb in VSCode though.

```{code-cell}
using PlotlyJS

f(x, y) = x^2 + 2y^2

x = 6

xlim=[-10, x]
ylim=[-10, 10]

xs = LinRange(xlim..., 101)
ys = LinRange(ylim..., 101)
zs = [f(x, y) for x in xs, y in ys]

y = 4
dy = 4
f_y(y) = 4y

traces = GenericTrace[]

push!(traces, PlotlyJS.surface(x=xs, y=ys, z=zs, 
        visible=true, showscale=false, opacity=0.8))
push!(traces, PlotlyJS.surface(x=[x, x+0.001], y=ylim, z=[[maximum(zs), minimum(zs)] [maximum(zs), minimum(zs)]], 
        visible=true, showscale=false, color="grey", opacity=0.3))
push!(traces, PlotlyJS.scatter3d(x=fill(x, size(ys)), y=ys, z=[x^2 + 2y^2 for y in ys], 
        visible=true, showlegend=false, mode="lines", line=attr(color="red", width=2)))

for y in ys
    push!(traces, PlotlyJS.scatter3d(x=fill(x, 2),y=[y-dy, y+dy], z=[f(x,y)-f_y(y)*dy, f(x,y)+f_y(y)*dy], 
            mode="lines", visible=false, showlegend=false, line=attr(color="orange", width=5)))
end

layout = Layout(
    sliders=[attr(
        steps=[
            attr(
                label=round(y, digits=2),
                method="update",
                args=[attr(visible=[fill(true, 3); fill(false, i-1); true; fill(false, 101-i)])]
            )
            for (i, y) in enumerate(ys)
        ],
        active = y,
        currentvalue_prefix="x = 6, y = ",
    )],
    scene = attr(
        xaxis = attr(range=[-10,10]),
        yaxis = attr(range=[-10,10]),
        zaxis = attr(range=[0,300])
    ),
)

p = PlotlyJS.plot(traces, layout)
```