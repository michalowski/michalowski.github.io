---
title: "Marek2D"
order: 15
excerpt: "A Rhino plugin for 2-D drawings from 3-D models: an alternative to Make2D, with glass and shadows."
header:
  image: /assets/images/marek2d/marek2d-header.png
  teaser: /assets/images/marek2d/marek2d-teaser.png
---

`Marek2D` is a plugin for Rhino 8 and later that turns a 3-D model into a 2-D line drawing of the current view, as an alternative to Rhino's built-in `Make2D` command.
It's free to use (search for `Marek2D` in the Package Manager), but YMMV, and feedback is welcome.

![Marek2D's drawing of a room cut by a section, with shadows, glass and furniture](/assets/images/marek2d/marek2d-room.png)

## Goals

`Make2D` is how most Rhino users get line drawings out of a model, and most of them spend time afterwards fixing what it got wrong and adding what it left out.
`Marek2D` sets out to make that cleanup unnecessary:

- **Correct lines.** Every line should be drawn where it is seen, hidden where it is hidden, and split exactly where it passes out of view. `Marek2D` checks each line against the 3-D model itself rather than trusting a single pass of Rhino's hidden-line engine, and it is tested against independently computed ground truth on hundreds of randomized scenes.
- **A drawing, not a starting point.** Sections with poché, objects seen through glass, shadows, the model's own hatches and text, and good first guesses for line weights and greyscale, so the result is close to ready for print or import to Illustrator.
- **Familiar.** It is used like `Make2D`: the same selection, the same kind of options with the same names where they mean the same thing, and the same layer conventions.
- **Fast enough.** The extra checking does take time, but the aim is to stay within an order of magnitude of `Make2D`'s time, with a progress bar, a time estimate, and `Esc` cancellability throughout.

In the comparisons below, black lines are visible, grey dashed lines are hidden, and solid grey lines are seen through glass.

## What it fixes

### Hidden lines at notches and inside corners

Where one object passes through a notch or recess in another, `Make2D` often gets the hidden lines wrong: edges shown that are hidden, or hidden that are in view.
In 16 of 20 randomized scenes of a bar through a notched panel, at least one of `Make2D`'s edges was wrong.
`Marek2D` splits each edge exactly where it passes behind something, and drew all 20 correctly.

![Make2D and Marek2D drawings of a bar passing through a notched panel](/assets/images/marek2d/marek2d-hidden-lines.png)

### Hidden edges drawn as visible, on plain boxes

Make2D's errors are not all on tricky geometry; they can be caused by an object's relative size or orientation to the camera.
In the test room further down, `Make2D` draws edges that are out of sight on simple objects.
On the bookshelf, the shelf at eye level is seen exactly edge-on, and `Make2D` draws two of its back edges as visible lines behind the front edge:

![Close-ups of Make2D's and Marek2D's drawings of a bookshelf's shelf seen edge-on](/assets/images/marek2d/marek2d-shelf.png)

On a chair seen from behind, `Make2D` mistakenly draws a hidden edge across the top corner of the chair's back:

![Close-ups of Make2D's and Marek2D's drawings of the top corner of a chair's back](/assets/images/marek2d/marek2d-chair.png)

### Missing lines where solids meet

Where one solid passes into another (a column into a slab, a beam into a wall, a pipe through a floor) their surfaces meet along a line that belongs to neither object, and `Make2D` doesn't draw it.
The usual fix is to manually generate those intersection edges.
`Marek2D` finds them while drawing, on both objects' layers, without adding anything to your model.

![Make2D and Marek2D drawings of a column passing through a ceiling slab](/assets/images/marek2d/marek2d-intersections-room.png)

### Text drawn over everything

`Make2D` copies text into the drawing whole, on top of everything, so a sign on a wall still shows through the crate standing in front of it.
`Marek2D` draws text as the outlines of its letters, at the size and angle it is shown, and hides it like any other line.

![Make2D and Marek2D drawings of a crate in front of a wall, each with text](/assets/images/marek2d/marek2d-text.png)

### And a few smaller ones

- A curve passing in front of a curved surface can come out hidden where it is in view (seen on a curve passing just in front of a cylinder's top).
- Clipping planes' include and exclude lists: `Make2D` applies them even when they are switched off, and ignores the sublayers of a listed layer. `Marek2D` follows what the viewport shows.
- `Make2D` leaves modeled hatches out of the drawing altogether. `Marek2D` carries them through to the drawing.

## What it adds

Most of the examples here can be demonstrated in a small test scene: a room cut open by a clipping plane, with a glazed door and windows, a tree outside, furniture, a plant, a floor hatch and the sun.

![Make2D and Marek2D drawings of the whole room](/assets/images/marek2d/marek2d-room-pair.png)

### Seeing through glass

`Make2D` treats glass as solid, so everything behind a window becomes a hidden line.
`Marek2D` puts what is seen through transparent objects on its own `Through_Glass` layer, to style separately (and on `Hidden` too, since it is behind something).
An object counts as glass if its material has some transparency, or a name with "glass", "clear", "window", "transparent" or "translucent" in it.

![Make2D and Marek2D drawings of a tree seen through a glazed door](/assets/images/marek2d/marek2d-glass.png)

### Shadows

With the sun on in Rhino's `Sun` panel, `Marek2D` shades every visible area in shadow, whether cast or self-shadowed, with closed outlines around each area.
The hatch is ready as plain lines for print or export, and also as one grouped hatch, so the pattern can be changed with one click.
Its lines run in the direction of the sunlight as the view shows it, so the long edges of cast shadows are lined by unbroken hatch lines.
In a section, the shadows come from the whole model before the cut by default, so a roof the section removes still shades the room below; or from the model as cut.

![Make2D and Marek2D drawings of a box, a cylinder and a sphere on a slab in sunlight](/assets/images/marek2d/marek2d-shadows.png)

### Sections with poché

Where an active clipping plane cuts a solid, the section line is drawn heavier on its own layer, and the cut faces are hatched, with sections that touch merged into one region.
Clipping planes are applied the way the viewport shows them: including hidden planes, depth-limited planes, and planes that cut only some objects.

### Hatches and text from the model

Hatches and text in the model are carried into the drawing as lines, hidden behind solids like anything else, without exploding anything in the model.
All hatches, the model's and `Marek2D`'s own, line up with the world origin, so neighboring hatches continue each other's lines.

![Make2D and Marek2D drawings of a corner of the room with a tiled floor](/assets/images/marek2d/marek2d-hatches.png)

### Meshes by their lines

Which of a mesh's edges `Make2D` draws is hard to predict: here it drew the faceted pot without its facets.
`Marek2D` draws a mesh's outline, its open edges and its creases (where faces meet at more than an angle you choose), so a smooth mesh comes out as a clean outline and a faceted one with its facets.
Meshes cast and receive shadows like solids, and can be cut by clipping planes.

![Make2D and Marek2D drawings of a plant with a faceted pot and smooth foliage](/assets/images/marek2d/marek2d-meshes.png)

### Layers you can work with

The drawing's layers mirror the model's, under one layer per kind of line, in a fixed order (sections on top, shadows at the bottom) that carries over to an Illustrator export.
Lines from layers named "Entourage" come out thinner, so people, trees and furniture sit back from the architecture.
A new drawing lands on top of the previous one, even after you've moved that one, so a redrawn view lines up with the last.

## How it works

`Make2D` hands the model to Rhino's hidden-line engine, which works out in one pass what is seen and draws it.
That engine is fast, but where it is wrong (at a notch, a shared edge, a curve grazing a surface) the drawing is wrong, and there is no second opinion.

`Marek2D` asks a more direct question, one line at a time.
It first prepares the model: cuts it at the clipping planes and caps the cuts, unpacks blocks, and turns hatches and text into curves.
It then collects every line the drawing could contain: the edges of every object, the outlines of curved surfaces, the lines where objects pass through each other, the section lines and the model's own curves.
For each line, it works out from the 3-D model where the line could pass behind something, and there fires a ray from the line towards the camera: if the ray hits a solid, that stretch is hidden; if it only passes through glass, it is seen through glass; otherwise it is in view.
The line is split at exactly those places.
Shadows are the same question asked from the sun's position: a point is in shadow when the ray towards the sun is blocked.

![How Marek2D makes a drawing, step by step, with the options that add time to each step](/assets/images/marek2d/marek2d-how.png)

It should be noted that the extra checking is slower than `Make2D`.
The test room takes about 11 seconds against `Make2D`'s 1.3; a 260-object house about 20 seconds against 2; a 22,000-object model about 15 minutes against under a minute.
We think that is a good tradeoff:

- The checking is what makes the lines right. A hidden-line error in a drawing is hard to spot and slow to fix by hand, and a single wrong line can say something false about the design.
- The same checking is what makes the rest possible. Glass is just a third answer to "what is in front of this line?", and shadows are the same question asked from the sun, so they come with the method rather than as separate, approximate passes.
- Seconds or minutes of the computer's time replace much longer stretches of tracing, retouching and redrawing by hand, every time the design changes and the drawing is made again.
- You can always see where it is: the command line gives a first estimate, the progress bar names the stage and the time left, and `Esc` stops it without leaving anything behind.

## Using it

### Installing

1. In Rhino, open the Package Manager (`PackageManager` command).
2. Search for `Marek2D` and install it.
3. Restart Rhino. The `Marek2D` command is then available; a toolbar button is in the Toolbars panel (hidden by default).

### Making a drawing

1. Set up the view to draw: any viewport, parallel or perspective. Clipping planes active in that view are applied.
2. For shadows, turn on the sun in the Sun panel.
3. Select the objects to draw (or run the command first and select them when asked).
4. Run `Marek2D`.
5. Change any options at the prompt, then press `Enter`.
6. Wait for the drawing: the command line shows an estimate and the progress. `Esc` cancels and removes everything drawn so far.
7. The drawing appears flat on the world XY plane, on top of the previous drawing (or at the origin), and is left selected, ready to move.
8. One `Undo` removes the whole drawing and its layers.

### Options

Each option is remembered between runs; defaults are shown in brackets.

- **`CreateHiddenLines`**: draw hidden lines.
  - `Yes`: hidden lines are drawn, dashed, on `Hidden`.
  - [`No`]: only what is seen.
- **`ShowTangents`**: edges between faces that meet smoothly (a fillet's edges).
  - `Yes`: drawn.
  - [`No`]: left out.
- **`CreateClippingPlaneIntersections`**: the view's clipping planes.
  - [`Yes`]: the model is cut as the view shows it, with section lines on `Clipping` and poché on `Poche hatch`.
  - `No`: clipping planes are ignored and the whole model is drawn.
- **`CreateObjectIntersections`**: lines where solids pass through each other.
  - [`Yes`]: drawn, on both objects' layers.
  - `No`: left out, as `Make2D` does.
- **`ThroughGlass`**: what is seen through transparent objects.
  - [`Yes`]: drawn on `Through_Glass` (and on `Hidden`).
  - `No`: glass hides what is behind it, as `Make2D` does.
- **`IncludeHatches`**: hatches in the model.
  - [`Yes`]: their pattern lines are drawn, hidden behind solids like any line.
  - `No`: left out, as `Make2D` does.
- **`IncludeText`**: text in the model.
  - [`Yes`]: drawn as the outlines of its letters, at the size shown, hidden behind solids like any line. Text set to face the view faces the drawing.
  - `No`: left out.
- **`RegisterWithPrevious`**: where the drawing goes.
  - [`Yes`]: where the previous `Marek2D` drawing is, following it if it has been moved.
  - `No`: its lower-left corner at the world origin.
- **`BlockClipping`**: how a clipping plane's include and exclude lists apply to blocks.
  - [`ByInstance`]: by the block instance and its layer, as `Make2D` does.
  - `ByMember`: by each object's own layer inside the block, as the viewport does.
- **`Shadows`**: shadows from the Sun panel's sun (none while it is off).
  - [`WholeGeometry`]: cast by the whole model, as if uncut, so a roof the section removes still shades the room.
  - `CutGeometry`: cast by the model as cut.
  - `Off`: no shadows.
- **`MeshCreaseAngle`** [`30`]: in degrees. On a mesh, an edge between faces meeting more sharply than this is drawn.
- **`LayerName`** [`Marek2D`]: the top-level layer for the drawing.

Small details need a model tolerance fine enough to resolve them: `Marek2D` warns when the tolerance is coarse for the geometry.

### The drawing's layers

Everything goes under one top-level layer (`Marek2D`, or the `LayerName` you choose).
Under each kind of line, sublayers reflect the model's own layer tree, so each object's lines end up on a layer named after its own; the tree below shows the top-level layers.
Layers that would be empty aren't made, and every layer starts turned on.

- **`Marek2D`**
  - **`Clipping`**: section lines, where clipping planes cut solids. Black, 0.5mm.
  - **`Poche hatch`**: the cut faces.
    - **`Unexploded`**: one hatch per region, all grouped so that you can click and change the pattern in one go.
    - **`Exploded`**: the same hatches as plain lines, for print and export. 0.125mm.
    - **`Boundaries`**: the outline of each cut region, merged where sections touch, to re-hatch or thicken as you like. 0.5mm.
  - **`Visible`**: everything that can be seen. Black, 0.25mm (0.125mm if under an "Entourage" sublayer).
  - **`Through_Glass`**: what is seen through glass. Grey, 0.25mm.
  - **`Hidden`**: hidden lines, when `CreateHiddenLines` is on. Grey, dashed, 0.125mm.
  - **`Shadow hatch`**: the areas in shadow, when the sun is on and `Shadows` is set.
    - **`Unexploded`**: one hatch per area, all grouped, with lines running along the sunlight.
    - **`Exploded`**: the same hatches as plain lines. Grey, 0.0625mm.
    - **`Boundaries`**: the closed outline of each shaded area, to re-hatch as you like.

Please email hello@michalow.ski with any questions or feedback.
