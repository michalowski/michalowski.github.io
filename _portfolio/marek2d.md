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

With the sun set up in the model, `Marek2D` adds closed curves around every visible area in shadow, whether cast or self-shadowed.
These areas are hatched for you, as plain lines ready for print or export, and also as a grouped hatch (on a layer that starts off) so the pattern can be changed with one click.
The hatch lines run in the direction of the sunlight as the view shows it, so the long edges of cast shadows are lined by unbroken hatch lines.
In a section, the shadows come from the whole model before the cut by default, so a roof the section removes still shades the room below, or from the model as cut.

![Make2D and Marek2D drawings of a box, a cylinder and a sphere on a slab in sunlight](/assets/images/marek2d/marek2d-shadows.png)

Also:
the output layers mirror the model's own layers, in a fixed order (top to bottom: sections, poché, visible, through glass, hidden, shadows), which carries over to an Illustrator export;
clipping planes are applied the way the viewport shows them, including hidden planes, caps to hide solid interiors, and include/exclude lists;
cut faces get poché hatching, with the sections merged where they touch;
all hatches use scale 1 and the world origin as their base point, so neighbouring hatches line up;
lines from layers named "Entourage" come out thinner;
hatches are exploded for the purposes of the drawing, but don't need to be exploded in your model;
and standalone curves and hatches are hidden behind solids like everything else.

The trade-off is speed.
`Marek2D` does much more checking than `Make2D`, so it is slower, noticeably so on large scenes.

## Using it

Select the objects to draw, make the view you want active (parallel or perspective), and run `Marek2D`.
Change any options, then press Enter.
The drawing is placed flat on the world XY plane, on layers under `Marek2D` (or a name of your choice, see `LayerName`), each with sublayers matching your model's layers.
It lands on top of the previous drawing, even if you've moved that one (see `RegisterWithPrevious`), and is left selected, so you can move it off at once.
The command line shows an estimate and the progress, and Esc cancels without leaving anything behind.
A toolbar button is available from the Toolbars panel (hidden by default).

Options, remembered between runs:

- **CreateHiddenLines** (No): also draw hidden edges, dashed, on `Hidden`.
- **ShowTangents** (No): draw the edges between smoothly joined faces.
- **CreateClippingPlaneIntersections** (Yes): draw section lines where active clipping planes cut solids, on `Clipping`, and poché the cut faces on `Poche hatch`: the merged outlines on `Boundaries`, the hatching as lines on `Exploded`, and as a grouped hatch on `Unexploded` (turned off).
- **CreateObjectIntersections** (Yes): draw the lines where solids pass through each other. Unlike with `Make2D`, you don't need to add these intersections to your model: they are generated temporarily, for the drawing only.
- **ThroughGlass** (Yes): draw what is seen through transparent materials on `Through_Glass`. An object counts as glass if its material has transparency or a name containing "glass", "clear", "window", "transparent" or "translucent".
- **IncludeHatches** (Yes): include hatch patterns in the drawing, occluded when behind solids like any other line. Unlike with `Make2D`, you don't need to explode hatches in your model: they are exploded temporarily, for the drawing only.
- **Shadows** (WholeGeometry): with the Sun panel's sun on, add the areas in shadow on `Shadow hatch`: the hatching as lines on `Exploded`, as a grouped hatch on `Unexploded`, and the areas' outlines on `Boundaries` (both turned off). `WholeGeometry` shades with the whole model, as if uncut; `CutGeometry` with the model as cut; `Off` draws no shadows. Cut faces are never shaded. With the sun off, no shadows are drawn whatever this is set to.
- **RegisterWithPrevious** (Yes): place the drawing where the previous `Marek2D` drawing is, following it if you've moved it, so a redrawn view lines up with the last one. `No` puts its lower-left corner at the world origin.
- **BlockClipping** (ByInstance): how a clipping plane's include/exclude lists apply to blocks. `ByInstance` matches the block instance and its layer, as `Make2D` does; `ByMember` matches each block member's own layer, as the viewport does.
- **LayerName** (Marek2D): the top-level layer for the drawing.

Small details need a fine enough model tolerance; `Marek2D` warns when the tolerance is coarse for the geometry.
Large models take a while; the estimate is refined as it runs.
