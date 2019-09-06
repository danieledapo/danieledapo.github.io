---
title: "Generating terrain mesh from noise functions"
date: 2019-08-29T21:52:25+02:00
---

Recently at work we bought a 3d printer and sure enought I wanted something
cool to print to try out the new toy. I didn't want to just search for ready to
print models online because that didn't sound fun. Instead, I figured I should
print something that I created.

Since I don't know how to use 3d modeling software (e.g. Blender) to create
good looking models, my only option left was to build a program able to
generate them.

With regard to what to generate, I've always been fascinated by those super
detailed models of terrain and landscapes and so I decided to create something
like that. This was the beginning of [terrain-mesh][terrain-mesh].


## When random is too random

I didn't want to write a program that was able to generate a single model only,
but I thought it was a good idea to have it generate different models each time
the program is run. This way I could choose which models I like the most and 3d
print them.

No problem I thought, after all pseudo random number generators are not rocket
science and they're quite easy to use. I'll just generate some numbers that
represent the height of the terrain at several points, connect those vertices
together and voilà. Here are the results

![random-terrain](/post/terrain-mesh/random-terrain.png)

Does it look like real terrain to you? I doubt so.

The problem is that pseudo random generators are too good and produce numbers
that really look unrelated to each other. In fact, if you ever tried to plot
some random numbers you will likely have seen something like

![random-plot](/post/terrain-mesh/random-plot.svg)

which looks pretty random, but doesn't look organic or realistic at all. At
this point I realized that I had to look for other ways to generate random
numbers that would look smooth when plotted in sequence.


## Noise functions

After a bit of searching online, I came across the concept of noise functions
which can be seen as pseudo random number generators that are guaranteed to be
smooth ad continuos. Exactly wat I needed!

Albeit conceptually similar to PRNGs, they're a bit different to use because
you need to provide a "coordinate" to calculate the random value at. Asking the
noise value at a given position will _always_ give you the same results (i.e.
the noise function is deterministic) but you can change the seed of the
generator to get new random results.

Since noise functions involve a position, it should come as no surprise that
there are noise functions for 1D, 2D, 3D and even infinite dimensions!

Here's a plot of [Perlin noise][perlin-noise] in 1D

![noise-plot](/post/terrain-mesh/noise-plot.svg)

This looks much more smooth and organic. If you squint hard enough you might
see the outline of a mountain or hill which is almost what I needed.


## Terrain meshes

At this point I had a way to generate smooth random values by using 2D Perlin
Noise, what was left to do was to actually create the faces and vertices of the
mesh. This revealed to be tricky, but not too hard.

The overall algorithm I use to generate terrain can be summerized as:

- generate a widthxdepth grid of 2d perlin noise values that represent the
  height of the terrain at those specific points;

- build the faces using quadrilaterals and not triangles because they're a bit
  easier to implement. Converting this quad mesh to a triangle mesh shouldn't
  be difficult, but I didn't bother to because most 3D software can handle quad
  meshes just fine;

- optionally add some support faces to the bottom and to the sides of the mesh
  which are necessary for 3D printing;

- save the mesh as an obj file.


Here's a rendering of a terrain created by `terrain-mesh`.

![terrain.png](/post/terrain-mesh/terrain.png)


## Conclusions

I'm quite satisfied with the terrains I'm able to generate and with their
diversity. Eventually I managed to 3d print some of these terrains, here's a
photo of two of them

![3d-printed.jpg](/post/terrain-mesh/3d-printed-small.jpg)

I think they're super cool and they have deserved a spot on my desk at work.
Next step is to use some real heightmaps to create the mesh of some real
terrain like the Moon or Mars. To be completely honest, I'm already able to do
so, but the way I'm doing so is too silly and the final meshes are not really
usable because they have way too many vertices and faces. Anyway, see you in a
while.


[terrain-mesh]: https://github.com/d-dorazio/terrain-mesh
[perlin-noise]: https://en.wikipedia.org/wiki/Perlin_noise
