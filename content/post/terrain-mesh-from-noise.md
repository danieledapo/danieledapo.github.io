---
title: "Generating terrain mesh from noise functions"
date: 2019-08-29T21:52:25+02:00
draft: true
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

![noise-plot](/post/terrain-mesh/noise-plot.svg)


## Terrain meshes

Explain a bit more how the terrain-mesh algorithm works. Especially the quads
faces.


## Conclusions

[terrain-mesh]: https://github.com/d-dorazio/terrain-mesh
