# Diagrams

Using [`Mermaid.js`](https://mermaid.js.org/) and [`sphinxcontrib-mermaid`](https://sphinxcontrib-mermaid-demo.readthedocs.io/en/latest/), one can create diagrams like the one below.

```{mermaid}

    sequenceDiagram
      participant Alice
      participant Bob
      Alice->John: Hello John, how are you?
```

There are some limitations to `mermaid.js`, for example one cannot always align nodes or position them in specific places.
Below is a quick attempt at reproducing Figure 1.2 from `optimisation-notes.pdf` that highlights some of the shortcomings.

```{mermaid}
flowchart BT
  m1(("$$t=1$$"))
  m2(("$$t=2$$"))
  m3(("$$\dots$$"))
  m4(("$$t=T$$"))
  m1 -->|"$$k_1$$"| m2 -->|"$$k_2$$"| m3 -->|"$$k_{T-1}$$"| m4
  t1["$$D_1$$"]
  t2["$$D_2$$"]
  t3["$$D_3$$"]
  t4["$$D_4$$"]
  b1 --> m1 --> t1
  b2 --> m2 --> t2
  b3 --> m3 --> t3
  b4 --> m4 --> t4
  b1["$$p_1$$"]
  b2["$$p_2$$"]
  b3["$$p_3$$"]
  b4["$$p_4$$"]

  linkStyle 0,1,2 opacity:0.5;

  classDef mid_node fill:orange,stroke:black,width:40px;
  class m1,m2,m3,m4 mid_node;
  classDef transparent_node fill:transparent,stroke:transparent;
  class t1,t2,t3,t4,b1,b2,b3,b4 transparent_node;
```

The math may not be rendering above, see {ref}`annoyance:mermaid-math`.

One can change the aesthetic attributes of the diagram elements, the above one contains examples of doing that within the diagram specification itself.
One can also create a theme and set it either for a specific diagram or make it the default option for all diagrams.
However, doing the former right now requires a workaround ([example here](https://github.com/mgaitan/sphinxcontrib-mermaid/issues/108#issuecomment-1486713225)), and I'm not sure if the latter will work all the time (for example when `mmdc` is involved).

## Further reading
- [`Mermaid.js` documentation](https://mermaid.js.org/intro/)
- Alternative diagram tools for Jupyter-Book [here](https://opencomputinglab.github.io/SubjectMatterNotebooks/diagram/sphinx-diagrammers.html)
- [Graphviz](https://graphviz.org/) seems to have a [native extension](https://www.sphinx-doc.org/en/master/usage/extensions/graphviz.html) for Sphinx