---
title: Spatial Data in D3
layout: post
output: 
  md_document:
    preserve_yaml: true
---

*Manipulating spatial data in D3*

1.  These notes summarize methods for static visualization of geospatial
    data in D3.

2.  Before we can make any plots, we need to be able to read in data. To
    read in vector data, it’s most convenient to store the data as
    geojson and read it in using `d3.geojson()`. The javascript object
    created by this function includes a `features` array. The `$i^{th}$`
    element in this array gives properties of the `$i^{th}$` feature in
    the vector dataset.

3.  For example, we can use this to read in the glaciers data.

4.  Even though these look like basic javascript objects, d3.geojson is
    actually keeping track geographic metadata behind the scenes. This
    allows us to do some basic geographic queries directly from
    javascript. For example, if we want to query the geographic centroid
    or areas (in square kilometers) of each feature, we can use the
    functions below.

    \[d3-geo query example\]

    A variety of related processing functions can be found in the
    `d3-geo` library.

5.  How can we display these data? We need a way of translating the
    abstract data into SVGs on the screen. For vector data, we can use
    D3’s geoPath generators. These work like line generators — they are
    initialized with visual encoding functions and can then be applied
    to any new collection of vector features.

6.  For example, we can use a path generator to draw the glacier
    boundaries.

7.  Remember that we have the vector feature attributes stored in
    `.features`. This allows us to modify the SVG attributes to reflect
    the data. For example, we can shade the glaciers in by their depth.

8.  We haven’t considered raster data in this lecture. This is because
    javascript doesn’t have a simple built-in way to handle raster data.
    If we want to visualize a raster dataset, we need to first convert
    them to simple PNG images — this loses the geographic metadata, and
    so any geographic processing has to be done before this step. In
    practice, it’s common to either manually convert to a PNG or to use
    a tiling library, which automatically converts raster data into a
    collection of PNGs. Both of these techniques are beyond the scope of
    our class, unfortunately.