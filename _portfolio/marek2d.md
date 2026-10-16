---
title: "Marek2D"
order: 15
excerpt: "A Rhino plugin for 2-D drawings from 3-D models: an alternative to Make2D, with glass and shadows."
header:
  image: /assets/images/marek2d/marek2d-header.png
  teaser: /assets/images/marek2d/marek2d-teaser.png
---

Marek2D is a Rhino 8 plugin that turns a 3-D model into a 2-D line drawing of the current view, as an alternative to Rhino's built-in Make2D command. It decides which edges are visible with its own occlusion test instead of trusting Rhino's, checked against independently computed ground truth on hundreds of randomized test scenes, and it adds a few things Make2D doesn't do. Free to use.

In each comparison below: black lines are visible, grey dashed lines are hidden, and solid grey lines are seen through glass.

### Hidden-line accuracy

Marek2D checks every edge against the model itself and splits it exactly where it passes out of view, including tricky cases like a bar running through a notch in a panel, as here, and edges that two objects share.

![Make2D and Marek2D drawings of a bar passing through a notched panel](/assets/images/marek2d/marek2d-hidden-lines.png)

### Intersection lines

Where one solid passes through another, Marek2D draws the line where their surfaces meet, on both objects' layers. Make2D leaves it out.

![Make2D and Marek2D drawings of a thin panel passing through a box](/assets/images/marek2d/marek2d-intersections.png)

### Seeing through glass

Make2D treats glass as solid, so everything behind a window becomes a hidden line. Marek2D puts what is seen through transparent objects on its own Through_Glass layer, to style separately.

![Make2D and Marek2D drawings of a box seen through a pane of glass](/assets/images/marek2d/marek2d-glass.png)

### Shadows

With the sun set up in the model, Marek2D adds closed curves around every visible area in shadow, cast or self-shadowed, already hatched, with the hatches grouped so the pattern can be changed with one click.

![Make2D and Marek2D drawings of a box, a cylinder and a sphere on a slab in sunlight](/assets/images/marek2d/marek2d-shadows.png)

Also: the output layers mirror the model's own layers; clipping planes are applied the way the viewport shows them, including hidden planes and include/exclude lists; and standalone curves and hatches are hidden behind solids like everything else.

The trade-off is speed. Marek2D does much more checking than Make2D, so it is slower, noticeably so on large scenes.
