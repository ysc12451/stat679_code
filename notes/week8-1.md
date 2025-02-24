---
title: Temporal Data in D3
layout: post
output: 
  md_document:
    preserve_yaml: true
---

*Static temporal data visualization in D3*

1.  These notes give the D3 analogs of some of the visualizations
    created in our earlier notes on temporal visualization. For a pure
    static visualization, these methods are not necessary — the R
    approach can give satisfactory results more easily. However, if we
    ever want to customize the appearance or interactivity beyond what
    is possible in the R packages discussed earlier, then these examples
    can serve as a starting point.

2.  Let’s begin with the basic line plot. We have had a few examples
    before
    \[[1](https://krisrs1128.github.io/stat679_notes/2022/06/01/week4-3.html),
    [2](https://krisrs1128.github.io/stat679_notes/2022/06/01/week6-4.html)\],
    but we swept important details about D3’s path generators under the
    rug. Remember that in the first of those examples, we had manually
    generated paths using strings like`M 100 100 L 200 105 L 300 115`.
    This means to start a path at pixel coordinates (100, 100), move to
    the right and down by (200, 105) pixels, and so on.

3.  To map data to these types of coordinates, we can use an SVG path
    generator. This is a function that converts an array of javascript
    objects to SVG path strings like the one above. This is accomplished
    by giving `d3.line()` functions that output the `x` and `y` pixel
    coordinates from objects representing individual timepoints. The `x`
    and `y` functions are usually light wrappers of scales that map the
    raw data into pixel coordinates.

        d3.line()
          .x([helper to get x pixel coordinate])
          .y([helper to get y pixel coordinate])

4.  For example, to regenerate the lynx plot, we first store the time
    series data into an array of objects,

        let data = [{Year: ..., Lynx: ...}, ...] 

    define a temporal scale for the x-axis,

         d3.scaleTime()
           .domain(x_extent)
           .range([margins.left, width - margins.right]),

    and finally apply a path generator defined usign `d3.line()`

        function draw_line(data, scales) {
          path_generator = d3.line()
            .x(d => scales.x(d.Year))
            .y(d => scales.y(d.Lynx));

          d3.select("#line")
            .selectAll("path")
            .data([data]).enter()
            .append("path")
            .attr("d", path_generator);
        }

    which together with some axis annotation functions creates this
    figure,

<iframe src="https://krisrs1128.github.io/stat679_code/examples/week8/week8-1/line.html" height="500" width="800">
</iframe>

1.  If we want to draw a collection of paths, we can use an array of
    arrays. Each element of the outer collection provides a path; each
    element within the inner arrays gives one timepoint.

        [
          [{t1, value1}, {t2, value2}, ...] // array for first line
          [{t2, value2}, {t2, value2}, ...] // array for second line
          ...
        ]

2.  For example, suppose we want to create a visualization of daily
    electricity demand over several months. We want each line to
    correspond to a single 24 hour period, and will have a few dozen
    lines.

<iframe src="https://krisrs1128.github.io/stat679_code/examples/week8/week8-1/electricity.html" height="500" width="700">
</iframe>

1.  At this point, it’s not hard to add in tooltips using mouseovers.
    For example, the implementation calls `update_series` every time the
    mouse is moved. The first block updates the `stroke` color by
    checking against the current line’s ID, which is stored in
    `ev.target.id`. The second block replaces the previous tooltip text
    and positions it at the mouseevent’s location.

<!-- -->

    function update_series(ev, data) {
      d3.select("#series")
        .selectAll("path")
        .attrs({
          stroke: d => d[0].Date_string == ev.target.id ? "red" : "#a8a8a8"
        })
        
      d3.select("#tooltip")
        .style("top", ev.clientY + "px")
        .style("left", (20 + ev.clientX) + "px")
        .text(ev.target.id)
    }

<iframe src="https://krisrs1128.github.io/stat679_code/examples/week8/week8-1/electricity-2.html" height="400" width="700">
</iframe>

1.  Path generators are just one example of a D3 function that maps raw
    data to more complex visual marks. For example, to create a stacked
    time series visualization, we can use `d3.stack()` together with
    `d3.area()`. `d3.stack()` reshapes the data so that the y-axis
    values for adjacent bands are easy to access.

        // toy input / output of d3.stack()

    while `d3.area()` converts the output from `d3.stack()` into a path
    generator analogous to `d3.line()`, but for full shaded areas rather
    than isolated lines.

        let area_generator = d3.area()
          .x(d => scales.x(d.data.issue))
          .y0(d => scales.y(d[0])) // lower envelope
          .y1(d => scales.y(d[1])) // upper envelope
          .curve(d3.curveBasis) // makes it smooth

2.  Using an alternative centering, the same area generator can be used
    to create a D3 streamgraph. For example, this code
    \[[1](https://github.com/krisrs1128/stat679_code/blob/main/examples/week8/week8-1/xmen.html),
    [2](https://github.com/krisrs1128/stat679_code/blob/main/examples/week8/week8-1/xmen.js)\]
    regenerates the x-men graph that we created last week.

<iframe src="https://krisrs1128.github.io/stat679_code/examples/week8/week8-1/xmen.html" height="400" width="500" style="text-align: center">
</iframe>

1.  Just to show how flexible D3 can be, let’s see how we can create two
    other types of temporal visualizations from scratch: Gantt charts
    and Bump charts.

2.  Gantt charts encode the start and end of discrete temporal events.
    They are often used to organize projects. They can be generated by
    appending rectangles at the right task (y) and time (x) coordinates.

3.  Bump charts are line plots that show how ranks evolve over time. We
    can create the smooth arcing lines using `.curve(d3.curveBumpX)`.

4.  We can customize the color depending on whether the rank has
    increased or decreased by storing the change in the data and
    encoding this using each path’s stroke attribute.

5.  You can see a full interactive version of this visualization at this
    link — we’ll be discussing this in our next lecture. Notice the use
    of the Voronoi mouseover, like what we discussed in our
    [interactivity
    notes](https://krisrs1128.github.io/stat679_notes/2022/06/01/week6-3.html).
    We’ll discuss some more techniques for temporal interaction in the
    next set of notes.
