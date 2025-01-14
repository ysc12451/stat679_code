---
title: Graph Interactivity I
layout: post
output: 
  md_document:
    preserve_yaml: true
---

*View interaction in graphs*

1.  Interactivity makes it possible to tinker with different views of a
    graph and get immediate feedback. By exploring a sequence of these
    views, it can be possible to build up a holistic understand of even
    very complex graphs.

2.  It’s helpful to think of graph interaction as falling into three
    categories, though the boundaries can often be fuzzy. From most
    superficial to most substantive, these are,

         * View interactivity: For a fixed mapping from data to visual marks, we alter
         the user’s view so that different regions become easier to study.
         * Encoding interactivity: We can change the visual encodings of a fixed
         collection of data based on user queries.
         * Data interactivity: We can allow the user to manipulate the data that appear
         in any given graph.

    In these notes, we’ll consider a few examples of view interactivity.
    Later, we’ll discuss encoding and data interactivity.

3.  A simple form of view interactivity is panning and zooming.
    Together, they can be used to change the center and extent of the
    user’s field of view. Even though these operations don’t require any
    complex redrawing of the graph, they allow a simple form of
    overview + detail interactivity. We can zoom out to view the overall
    graph and then pan and zoom to specific neighborhoods of interest.

4.  In D3, panning and zooming can be implemented using `d3.drag()` and
    `d3.zoom()`, respectively. These are used to construct functions
    that can then be called on `g` elements containing the objects to
    pan and zoom over. For example, to pan and zoom over a simple set of
    circles, we can use this block,

5.  Application to the graph context works similarly. Here is an example
    where we can pan and zoom across a node-link diagram (can you think
    of how to do this for an adjacency matrix view?)

6.  There are more subtle forms of view interactivity. One interesting
    example discussed in the reading for this week is “edge lensing.”
    This type of interaction is designed to solve the problem of highly
    overlapping edges in dense regions of the graph. For example,
    suppose we want to identify the neighbors of a node that lies in the
    core of the graph. Since it lies in a dense region, there is a good
    chance that many links cross over it, even if they are not direct
    neighbors.

7.  The idea of the edge lens interaction is to create a “lens” that
    hides edges that are not directly relevant to the queried region.
    For example, this removes long-range interactions between nodes far
    from the lens.

8.  We can implement a simple version of this in D3. We can easily draw
    a lens by asking a circle to follow our mouse using a mousemove
    interaction.

9.  Next, we find all the nodes that are contained within the lens. In
    the updated view, we redraw edges for these nodes, and we make sure
    they lie above the lens.

10. This is not the most efficient implementation, since we draw the
    edges within the lens twice (both above and below the lens). In
    principle, we could compute the edge / lens intersections and change
    the line endpoints as we move. However, the resulting code would be
    harder for a new reader to understand, and except in the most
    compute-constrained environments, we should prefer readable
    implementations (this is in the spirit of “premature optimization is
    the root of all evil”).

11. There are a few other forms of lens-based view interactions. A
    variant of edge lenses brings all of a node’s neighbors into the
    currently hovered view. Fisheye lens are used to distort the view so
    that the lensed area gets expanded. The user’s past history of
    inputs can be used to define an “interest” function over regions of
    the graph, and areas considered more interesting can be expanded to
    take up more space in the layout.

12. None of these approaches directly change the graph data (Data
    Interactivity) or their encodings (Encoding Interactivity). In the
    next set of notes, we’ll consider these more substantive
    interactions.
