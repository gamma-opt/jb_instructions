---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.2
kernelspec:
  display_name: Julia 1.10.3
  language: julia
  name: julia-1.10
---

# Gradient-based optimization

## Linear vs Nonlinear

In general, optimization problems look like

```{math}
:label: opt_problem
:nowrap:

\begin{align*}
\mini &f(x) \\
\stf &g_i(x)\leq 0, i=1,\dots,m \\
&h_i(x)=0, i=1,\dots,l \\
&x\in X.
\end{align*}
```

Depending on the properties of {math}`f, g, h`, one can use {doc}`linear programming <lp_simplex>` or {doc}`MILP <mip_branch_cut>` methods.
In this page, we will focus on problems where {math}`f` is nonlinear, and how one can optimize such functions in the presence of constraints or not.

Even if {math}`f` is not linear, we will assume some structure about it, specifically differentiability.
In absence of any structure, efficient optimization may be very difficult (but not necessarily impossible, via various derivative-free methods like genetic algorithms).


Intuitively, a function is differentiable when a linear function can approximate it well in any given locality.
In the case of scalar-valued functions, this means lines.

```{code-cell} julia
:tags: ["remove-input"]

using WGLMakie, Bonito
Page(exportable=true, offline=true)

x = range(-π, π, 100)
xlims = (-π, π)

app = App() do session
    slider = Slider(x)
    fig, ax, lplot = lines(x, sin.(x); linewidth = 3)
    xlims!(ax, xlims)

    p = @lift(Point($slider[], sin($slider[])))
    splot = scatter!(ax, p; color = 2, colormap = :tab10, colorrange = (1, 10))

    slope = cos(slider[])
    intercept = sin(slider[]) - slope*slider[]
    abplot = ablines!(ax, [intercept], [slope]; color = 3, colormap = :tab10, colorrange = (1, 10))

    onjs(session, slider.value, js"""function on_update(new_val) {
        $(splot).then(plots=>{
            const scatter = plots[0]
            scatter.geometry.attributes.pos.array[0] = new_val
            scatter.geometry.attributes.pos.array[1] = Math.sin(new_val)
            scatter.geometry.attributes.pos.needsUpdate = true
        })
    }
    """)
    onjs(session, slider.value, js"""function on_update(new_val) {
        const slope = Math.cos(new_val);
        const intercept = Math.sin(new_val) - slope*new_val;
        const start_y = slope*(-Math.PI) + intercept;
        const end_y = slope*Math.PI + intercept;

        $(abplot).then(plots=>{
            const abplot = plots[0]
            // change the y coord for the (start/end)points of the line
            abplot.geometry.attributes.linepoint_end.data.array[3] = start_y
            abplot.geometry.attributes.linepoint_end.data.array[5] = end_y
            abplot.geometry.attributes.linepoint_end.needsUpdate = true
        })
    }
    """)

    return DOM.div(slider, fig)
end
```

Or in the case of two variables, the linear function becomes a plane.

```{code-cell} julia
:tags: ["remove-input"]

f(x,y) = -x^2-y^2
z = [f(i,j) for i in x, j in x]

app = App() do session
    x_slider = Slider(x)
    y_slider = Slider(x)
    fig, ax, plot = surface(x, x, z)

    p = @lift(Point($x_slider[], $y_slider[], f($x_slider[], $y_slider[])))
    splot = scatter!(ax, p; color = 2, colormap = :tab10, colorrange = (1, 10))

    z2 = [0 for i in x, j in x]
    surplot = surface!(ax, x, x, z2)

    evaljs(session, js"""
        const observables = $([x_slider.value, y_slider.value])
        const f = (x,y) => -(x**2)-(y**2)
        const der = (x) => -2*x

        function update(args) {
            const [x, y] = args;
            $(splot).then(plots=>{
                const scatter = plots[0]
                scatter.geometry.attributes.pos.array[0] = x
                scatter.geometry.attributes.pos.array[1] = y
                scatter.geometry.attributes.pos.array[2] = f(x,y)
                scatter.geometry.attributes.pos.needsUpdate = true
            });
            $(surplot).then(plots=>{
                const surface = plots[0]
                for (let i = 0; i<=10000; i++){
                    const x0 = surface.geometry.attributes.position.array[3*i]
                    const y0 = surface.geometry.attributes.position.array[3*i+1]
                    surface.geometry.attributes.position.array[3*i+2] = der(x)*(x0-x) + der(y)*(y0-y) + f(x,y)
                    surface.geometry.attributes.position.needsUpdate = true
                }
            });
        }
        Bonito.onany(observables, update)
        update(observables.map(x=> x.value))
        """)
    
    return DOM.div(x_slider, y_slider, fig)
end
```

Not every function is differentiable, and there can be different reasons why.
On the left plot below, we have a function that is undefined below {math}`x<0`, so it is not well-defined how one could approximate it at the point {math}`x=0`.
On the right, the function is continuous, but the approximation at {math}`x=0` changes significantly if one is approaching from the left or the right.

```{code-cell} julia
:tags: ["remove-input"]
using CairoMakie
CairoMakie.activate!()

fig = Figure(size = (1200, 400))

ax1 = Axis(fig[1,1], limits = (xlims, nothing))
ax2 = Axis(fig[1,2], limits = (xlims, nothing))

lines!(ax1, x[51:end], repeat([1], 50); linewidth = 3)
lines!(ax2, x, abs.(x); linewidth = 3)

fig
```

Differentiability of a function {math}`f` is important because it allows one access to its gradient {math}`\nabla f`, which at any point gives the direction (and rate) of fastest increase.
This is very useful information when one is maximizing an objective function.
When minimizing, we can still use the gradient since {math}`g=-f` will have {math}`\nabla g=-\nabla f`, implying that the negative gradient is the direction of steepest descent.

## Convex vs Nonconvex

The usefulness of the gradient begs a question. 
Suppose we are maximizing a function {math}`f` and we are at a point {math}`x` such that {math}`\nabla f(x)=0`.
What does this mean?
Since the direction of fastest increase is 0, is {math}`x` the maximum we were seeking?

The answer to this question depends on whether the problem in {eq}`opt_problem` is convex or not.
A convex problem is one where the objective is a convex function and the feasible region imposed by the constraints is a convex set.

TODO: Add explanation/illustration

Convexity effectively means that local information can be used globally and that every local optimum is also a global optimum.

## Gradient descent and variants

Adapted from [Emilien Dupont's code.](https://emiliendupont.github.io/2018/01/24/optimization-visualization/).

% This requires `d3.v4.js `, which is currently provided via _config.yml.
```{raw} html
<body>
<div id="d3-gd"></div>
</body>
<style>
.sgd {
    stroke: black;
}

.momentum {
    stroke: blue;
}

.rmsprop {
    stroke: red;
}

.adam {
    stroke: green;
}

.SGD {
    fill: black;
}

.Momentum {
    fill: blue;
}

.RMSProp {
    fill: red;
}

.Adam {
    fill: green;
}

circle:hover {
  fill-opacity: .3;
}
</style>
<body>
<script src="https://d3js.org/d3.v7.min.js"></script>
<script src="https://d3js.org/d3-contour.v1.min.js"></script>
<script src="https://d3js.org/d3-scale-chromatic.v1.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/13.0.0/math.min.js"></script>
<script>

/* Constants */

const width = 800,
    height = 500,
    nx = parseInt(width / 5), // grid sizes
    ny = parseInt(height / 5),
    h = 1e-7, // step used when approximating gradients
    drawing_time = 30; // max time to run optimization

// Parameters describing where function is defined
const domain_x = [-2, 2],
    domain_y = [-2, 2],
    domain_f = [-2, 8],
    contour_step = 0.5; // Step size of contour plot

const scale_x = d3.scaleLinear()
                .domain([0, width])
                .range(domain_x);

const scale_y = d3.scaleLinear()
                .domain([0, height])
                .range(domain_y);

const thresholds = d3.range(domain_f[0], domain_f[1], contour_step);

const contours = d3.contours()
    .size([nx, ny])
    .thresholds(thresholds);

const color_scale = d3.scaleLinear()
    .domain(d3.extent(thresholds))
    .interpolate(function() { return d3.interpolateYlGnBu; });

/* Value of f at (x, y) */
function f(x, y) {
    return -2 * Math.exp(-((x - 1) * (x - 1) + y * y) / .2) + -3 * Math.exp(-((x + 1) * (x + 1) + y * y) / .2) + x * x + y * y;
}

/* Returns gradient of f at (x, y) */
function grad_f(x,y) {
    let grad_x = (f(x + h, y) - f(x, y)) / h
        grad_y = (f(x, y + h) - f(x, y)) / h
    return [grad_x, grad_y];
}

function hess_f(x,y) {
    const xx = (f(x + h, y) - 2*f(x, y) + f(x - h, y)) / (h * h),
          xy = (f(x + h, y + h) - f(x, y + h) - f(x + h, y) + f(x, y)) / (h * h),
          yy = (f(x, y + h) - 2*f(x, y) + f(x, y - h)) / (h * h);
    return [[xx, xy], [xy, yy]];
}


/* Returns values of f(x,y) at each point on grid as 1 dim array. */
function get_f_values(nx, ny) {
    let grid = new Array(nx * ny);
    for (i = 0; i < nx; i++) {
        for (j = 0; j < ny; j++) {
            let x = scale_x( parseFloat(i) / nx * width ),
                y = scale_y( parseFloat(j) / ny * height );
            // Set value at ordering expected by d3.contour
            grid[i + j * nx] = f(x, y);
        }
    }
    return grid;
}

const f_values = get_f_values(nx, ny);

function draw_contour(selector, mousedown_fn) {
    const svg = d3.select(selector)
                  .append("svg")
                  .attr("width", width)
                  .attr("height", height);

    const function_g = svg.append("g").on("mousedown", mousedown_fn);

    function_g.selectAll("path")
            .data(contours(f_values))
            .enter().append("path")
            .attr("d", d3.geoPath(d3.geoIdentity().scale(width / nx)))
            .attr("fill", function(d) { return color_scale(d.value); })
            .attr("stroke", "none");
    
    return svg;
}

function create_menu(svg, labels) {
    const menu_g = svg.append("g");
    let draw_state = Object.fromEntries(labels.map(x => [x, true]));

    function press () {
        let type = d3.select(this).attr("class");
        if (draw_state[type]) {
            d3.select(this).attr("fill-opacity", 0);
            draw_state[type] = false;
        } else {
            d3.select(this).attr("fill-opacity", 0.5)
            draw_state[type] = true;
        }
    }

    menu_g.append("rect")
        .attr("x", 0)
        .attr("y", height - 40)
        .attr("width", width)
        .attr("height", 40)
        .attr("fill", "white")
        .attr("opacity", 0.2);

    menu_g.selectAll("circle")
        .data(labels)
        .enter()
        .append("circle")
        .attr("cx", function(d,i) { return width/labels.length  * (i + 0.25);} )
        .attr("cy", height - 20)
        .attr("r", 10)
        .attr("stroke-width", 0.5)
        .attr("stroke", "black")
        .attr("class", function(d) { console.log(d); return d;})
        .attr("fill-opacity", 0.5)
        .attr("stroke-opacity", 1)
        .on("mousedown", press);

    menu_g.selectAll("text")
        .data(labels)
        .enter()
        .append("text")
        .attr("x", function(d,i) { return width/labels.length * (i + 0.25) + 18;} )
        .attr("y", height - 14)
        .text(function(d) { return d; })
        .attr("text-anchor", "start")
        .attr("font-family", "Helvetica Neue")
        .attr("font-size", 15)
        .attr("font-weight", 200)
        .attr("fill", "white")
        .attr("fill-opacity", 0.8);

    return draw_state;
}

// these functions should accept x0, y0, plus anything more specified here
const gradient_container = {
    "SGD": [get_sgd_path, 2e-2, 500],
    "Momentum": [get_momentum_path, 1e-2, 200, 0.8],
    "RMSProp": [get_rmsprop_path, 1e-2, 300, 0.99, 1e-6],
    "Adam": [get_adam_path, 1e-2, 100, 0.7, 0.999, 1e-6]
}

function create_interactive_plot(selector, method_container) {
    const svg = draw_contour(selector, mouse_fn);
    let draw_state = create_menu(svg, Object.keys(method_container));
    const path_g = svg.append("g");

    function mouse_fn(event) {
        const point = d3.pointer(event);
        minimize(scale_x(point[0]), scale_y(point[1]));
    }

    function minimize(x0,y0) {
        path_g.selectAll("path").remove();
        for (const [name, [f, ...rest]] of Object.entries(method_container)){
            if (draw_state[name]) {
                let data = f(x0, y0, ...rest);
                draw_path(data, name.toLowerCase(), path_g)
            }
        }
    }
}

create_interactive_plot("#d3-gd", gradient_container);


/*
 * Set up optimization/gradient descent functions.
 * SGD, Momentum, RMSProp, Adam.
 */

function get_sgd_path(x0, y0, learning_rate, num_steps) {
    let sgd_history = [{"x": scale_x.invert(x0), "y": scale_y.invert(y0)}];
    let x1, y1, gradient;
    for (i = 0; i < num_steps; i++) {
        gradient = grad_f(x0, y0);
        x1 = x0 - learning_rate * gradient[0]
        y1 = y0 - learning_rate * gradient[1]
        sgd_history.push({"x" : scale_x.invert(x1), "y" : scale_y.invert(y1)})
        x0 = x1
        y0 = y1
    }
    return sgd_history;
}

function get_momentum_path(x0, y0, learning_rate, num_steps, momentum) {
    let v_x = 0,
        v_y = 0;
    let momentum_history = [{"x": scale_x.invert(x0), "y": scale_y.invert(y0)}];
    let x1, y1, gradient;
    for (i=0; i < num_steps; i++) {
        gradient = grad_f(x0, y0)
        v_x = momentum * v_x - learning_rate * gradient[0]
        v_y = momentum * v_y - learning_rate * gradient[1]
        x1 = x0 + v_x
        y1 = y0 + v_y
        momentum_history.push({"x" : scale_x.invert(x1), "y" : scale_y.invert(y1)})
        x0 = x1
        y0 = y1
    }
    return momentum_history
}

function get_rmsprop_path(x0, y0, learning_rate, num_steps, decay_rate, eps) {
    let cache_x = 0,
        cache_y = 0;
    let rmsprop_history = [{"x": scale_x.invert(x0), "y": scale_y.invert(y0)}];
    let x1, y1, gradient;
    for (i = 0; i < num_steps; i++) {
        gradient = grad_f(x0, y0)
        cache_x = decay_rate * cache_x + (1 - decay_rate) * gradient[0] * gradient[0]
        cache_y = decay_rate * cache_y + (1 - decay_rate) * gradient[1] * gradient[1]
        x1 = x0 - learning_rate * gradient[0] / (Math.sqrt(cache_x) + eps)
        y1 = y0 - learning_rate * gradient[1] / (Math.sqrt(cache_y) + eps)
        rmsprop_history.push({"x" : scale_x.invert(x1), "y" : scale_y.invert(y1)})
        x0 = x1
        y0 = y1
    }
    return rmsprop_history;
}

function get_adam_path(x0, y0, learning_rate, num_steps, beta_1, beta_2, eps) {
    let m_x = 0,
        m_y = 0,
        v_x = 0,
        v_y = 0;
    let adam_history = [{"x": scale_x.invert(x0), "y": scale_y.invert(y0)}];
    let x1, y1, gradient;
    for (i = 0; i < num_steps; i++) {
        gradient = grad_f(x0, y0)
        m_x = beta_1 * m_x + (1 - beta_1) * gradient[0]
        m_y = beta_1 * m_y + (1 - beta_1) * gradient[1]
        v_x = beta_2 * v_x + (1 - beta_2) * gradient[0] * gradient[0]
        v_y = beta_2 * v_y + (1 - beta_2) * gradient[1] * gradient[1]
        x1 = x0 - learning_rate * m_x / (Math.sqrt(v_x) + eps)
        y1 = y0 - learning_rate * m_y / (Math.sqrt(v_y) + eps)
        adam_history.push({"x" : scale_x.invert(x1), "y" : scale_y.invert(y1)})
        x0 = x1
        y0 = y1
    }
    return adam_history;
}


/*
 * Functions necessary for path visualizations
 */

const line_function = d3.line()
                      .x(function(d) { return d.x; })
                      .y(function(d) { return d.y; });

function draw_path(path_data, type, path_g) {
    const gradient_path = path_g.selectAll(type)
                        .data(path_data)
                        .enter()
                        .append("path")
                        .attr("d", line_function(path_data.slice(0,1)))
                        .attr("class", type)
                        .attr("stroke-width", 3)
                        .attr("fill", "none")
                        .attr("stroke-opacity", 0.5)
                        .transition()
                        .duration(drawing_time)
                        .delay(function(d,i) { return drawing_time * i; })
                        .attr("d", function(d,i) { return line_function(path_data.slice(0,i+1));})
                        .remove();

    path_g.append("path")
                   .attr("d", line_function(path_data))
                   .attr("class", type)
                   .attr("stroke-width", 3)
                   .attr("fill", "none")
                   .attr("stroke-opacity", 0.5)
                   .attr("stroke-opacity", 0)
                   .transition()
                   .duration(path_data.length * drawing_time)
                   .attr("stroke-opacity", 0.5);
}

/*
 * Start minimization from click on contour map
 */



</script>
```

## Newton's Method

Need to debug BFGS

```{raw} html
<body>
<div id="d3-newton"></div>
</body>
<style>
.newton {
    stroke: black;
}
.Newton {
    fill: black;
}
.bfgs {
    stroke: blue;
}
.BFGS {
    fill: blue;
}
</style>
<script>

const newton_container = {
    "Newton": [get_newton_path, 300, 1e-4],
    "BFGS": [get_bfgs_path, 100, 1e-4]
}
create_interactive_plot("#d3-newton", newton_container);

function golden_ls(f, a, b, l) {
    const alpha = 0.618 // 1/golden_ratio
    let lam = a + (1-alpha)*(b-a)
    let mu = a + alpha*(b-a)

    let fmu = f(mu)
    let flam = f(lam)
    while (b-a > l) {
        if (flam > fmu) {
            a = lam
            lam = mu
            flam = fmu
            mu = a + alpha*(b-a)
            fmu = f(mu)
        }
        else {
            b = mu
            mu = lam
            fmu = flam
            lam = a+(1-alpha)*(b-a)
            flam = f(lam)
        }
    }
    return (a+b)/2
}

function get_newton_path(x0, y0, num_steps, tol) {
    let newton_history = [{"x": scale_x.invert(x0), "y": scale_y.invert(y0)}];
    let x1, y1, gradient, hessian, lr;
    for (i = 0; i < num_steps; i++) {
        gradient = grad_f(x0, y0);
        if (math.norm(gradient) < tol) return newton_history;
        hessian = hess_f(x0, y0);
        dir = math.chain(hessian).inv().multiply(gradient, -1).done()
        const foo = (l) => f(x0 + l*dir[0], y0 + l*dir[1])
        lr = golden_ls(foo, 0, 10, 1e-7)
        x1 = x0 + lr * dir[0]
        y1 = y0 + lr * dir[1]
        newton_history.push({"x" : scale_x.invert(x1), "y" : scale_y.invert(y1)})
        x0 = x1
        y0 = y1
    }
    return newton_history;
}

function get_bfgs_path(x0, y0, num_steps, tol) {
    let bfgs_history = [{"x": scale_x.invert(x0), "y": scale_y.invert(y0)}];
    let x1, y1, gradient, s, g, rho;
    const id_matrix = math.matrix([[1.,0.],[0.,1.]]);
    let H = id_matrix;
    let next_grad = grad_f(x0, y0);
    for (i = 0; i < num_steps; i++) {
        gradient = next_grad;
        if (math.norm(gradient) < tol) return bfgs_history;
        dir = math.chain(H).multiply(gradient, -1).done();
        const foo = (l) => f(x0 + l*dir.get([0]), y0 + l*dir.get([1]));
        lr = golden_ls(foo, 0, 10, 1e-7);
        x1 = x0 + lr * dir.get([0]);
        y1 = y0 + lr * dir.get([1]);
        bfgs_history.push({"x" : scale_x.invert(x1), "y" : scale_y.invert(y1)});
        s = math.matrix([[x1-x0, y1-y0]]);
        next_grad = grad_f(x1, y1);
        g = math.matrix([math.subtract(next_grad, gradient)]);
        rho = math.chain(g).multiply(math.transpose(s)).inv().done().get([0,0]);
        H = math.add(
                math.multiply(
                    math.subtract(id_matrix, math.multiply(rho, math.transpose(s), g)),
                    H,
                    math.subtract(id_matrix, math.multiply(rho, math.transpose(g), s))
                ),
                math.multiply(rho, math.transpose(s), s)
        );
        x0 = x1;
        y0 = y1;
    }
    return bfgs_history;
}
</script>
```
