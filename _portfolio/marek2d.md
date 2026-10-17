---
title: "Marek2D"
order: 15
excerpt: "A Rhino plugin for 2-D drawings from 3-D models: an alternative to Make2D, with glass and shadows."
header:
  image: /assets/images/marek2d/marek2d-header.png
  teaser: /assets/images/marek2d/marek2d-teaser.png
---

`Marek2D` is a Rhino 8 plugin that turns a 3-D model into a 2-D line drawing of the current viewport, as an alternative to Rhino's built-in `Make2D` command.
It decides which edges are visible with its own occlusion test instead of trusting Rhino's, checked against independently computed ground truth on hundreds of randomized test scenes, and it adds a few things `Make2D` doesn't do.
It's free to use (search for `Marek2D` in the PackageManager), but YMMV, and feedback is welcome.

In each comparison below: black lines are visible, grey dashed lines are hidden, and solid grey lines are seen through glass.

## Hidden-line accuracy

`Marek2D` checks every edge against the model itself and splits it exactly where it passes out of view, including tricky cases like a bar running through a notch in a panel, as here, and edges that two objects share.

![Make2D and Marek2D drawings of a bar passing through a notched panel](/assets/images/marek2d/marek2d-hidden-lines.png)

## Intersection lines

Where one solid passes through another, Marek2D draws the line where their surfaces meet, on both objects' layers.
(`Make2D` leaves it out.)
These computed intersections are ephemeral for the purposes of the drawing, and don't need to be stored in your model.

![Make2D and Marek2D drawings of a thin panel passing through a box](/assets/images/marek2d/marek2d-intersections.png)

## Seeing through glass

`Make2D` treats glass as solid, so everything behind a window becomes a hidden line.
`Marek2D` puts what is seen through transparent objects on its own `Through_Glass` layer, for styling separately.

![Make2D and Marek2D drawings of a box seen through a pane of glass](/assets/images/marek2d/marek2d-glass.png)

## Shadows

With the sun set up in the model, `Marek2D` optionally adds closed curves around every visible area in shadow, whether cast or self-shadowed.
These areas are hatched for you, and grouped so the pattern can be changed with one click.

![Make2D and Marek2D drawings of a box, a cylinder and a sphere on a slab in sunlight](/assets/images/marek2d/marek2d-shadows.png)

Also:
the output layers mirror the model's own layers;
clipping planes are applied the way the viewport shows them, including hidden planes, caps to hide solid interiors, and include/exclude lists;
hatches are exploded for the purposes of the drawing, but don't need to be exploded in your model;
and standalone curves and hatches are hidden behind solids like everything else.

The trade-off is speed.
`Marek2D` does much more checking than `Make2D`, so it is slower, noticeably so on large scenes.

## Using it

Select the objects to draw, make the view you want active (parallel or perspective), and run `Marek2D`.
Change any options, then press Enter.
The drawing is placed flat at the world origin, on layers under `Marek2D` (or a name of your choice, see `LayerName`), each with sublayers matching your model's layers.
The command line shows an estimate and the progress, and Esc cancels without leaving anything behind.
A toolbar button is available from the Toolbars panel (hidden by default).

Options, remembered between runs:

- **CreateHiddenLines** (No): also draw hidden edges, dashed, on `Hidden`.
- **ShowTangents** (Yes): draw the edges between smoothly joined faces.
- **CreateClippingPlaneIntersections** (Yes): draw section lines where active clipping planes cut solids, on `Clipping`.
- **CreateObjectIntersections** (Yes): draw the lines where solids pass through each other. Unlike with `Make2D`, you don't need to add these intersections to your model: they are generated temporarily, for the drawing only.
- **ThroughGlass** (Yes): draw what is seen through transparent materials on `Through_Glass`. An object counts as glass if its material has transparency or a name containing "glass", "clear", "window", "transparent" or "translucent".
- **IncludeHatches** (Yes): include hatch patterns in the drawing, occluded when behind solids like any other line. Unlike with `Make2D`, you don't need to explode hatches in your model: they are exploded temporarily, for the drawing only.
- **Shadows** (No): with the Sun panel's sun on, add the areas in shadow: grouped hatches on `Shadows::Hatch`, with their boundary curves on `Shadows::Boundaries` (turned off).
- **BlockClipping** (ByInstance): how a clipping plane's include/exclude lists apply to blocks. `ByInstance` matches the block instance and its layer, as `Make2D` does; `ByMember` matches each block member's own layer, as the viewport does.
- **LayerName** (Marek2D): the top-level layer for the drawing.

Small details need a fine enough model tolerance; `Marek2D` warns when the tolerance is coarse for the geometry.
Large models take a while; the estimate is refined as it runs.
