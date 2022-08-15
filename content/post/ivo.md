---
title: "Ivo, the isometric voxel renderer!"
date: 2022-08-14T11:06:09+02:00
draft: true
---


It's been a long time since my last post, but I didn't stop doing things. This time I want to talk about [ivo][ivo-github], my isometric voxel renderer.

I've always been inspired by [@wblut][wblut]'s work, it's a great combination
between coding skills and art. In particular, his isometric pieces are
absolutely stunning, intricate and coherent at the same time. Just check his
twitter profile to have an idea of the complexity of his work!

I decided to try and come up with a similar system mainly to understand how it
worked. Also, since I recently got a pen plotter (yay!) I wanted the system to
produce pen-plotter friendly SVGs.


## The system

The system at its core is just a collection of voxels that can be turned on and
off programmatically, perform hidden line removal and ultimately save the set of
visible lines onto the SVG. Since I'm using the system mostly to plot the SVGs
I'm not interested in doing some cool shading and fills, I just wanted the
visible outlines of the voxels.

I could have tried to use some framework of some sort because voxel renderers
are quite common and useful, but I didn't find one capable of doing hidden line
removal and outputting SVGs. Hence, I decided to build everything from scratch.
At the end of the day, how hard can it be to take a set of voxels, project them
always in the same way and understand which lines are visible and which are not,
right?

Turns out it took me quite some time, more that I'd like to admit. This is the
main reason I'm writing this post, maybe someone someday will find this useful
and hopefully avoid a lot of frustration.


## Edge visibility

The first thing to notice is that with an isometric projection only three faces
can be visible. This simplifies things a bit because the renderer has to keep
track of those three faces only and can cut corners if need be. I chose to keep
track of the front, top and right faces, but the choice is completely arbitrary.

<img src="isocube.svg" alt="isometric cube" class="image-centered">


TODO: explain why we triangulate and how we calculate the visibility of the edges

<img src="triangulation.svg" alt="voxel triangulation" class="image-centered">


-- Triangulation, why?

The main idea of the rendering algorithm is to triangulate the faces of each
voxel and then check each edge of the triangle; if it's shared between at least
two adjacent voxels then the edge is invisible, otherwise it is visible.



- triangulate the top, front and right quadrilateral faces only since with an
  isometric projection those are the only visible faces. However, we must also
  keep track of which edges are invisible and which are actually visible (e.g.
  the diagonals of a quadrilateral should be invisible)
- for each edge of every triangle find out if there is more than one voxel
  sharing that edge with the same `nearness()` value. If there are then the edge is invisible, otherwise is visible.



## 3D to 2D projection

Now that we found out which edges are visible, they have to be projected from
the 3D coordinate space into the 2D isometric one. The way I do it is to first
project the 3D point into a regular 2D orthogonal space by removing the z
component.

At this point, the point in the 2D orthogonal plane must be projected into the
final isometric space. This space is actually just a grid of hexagons which
means it's made by two axis at 30°. Hence, the orthogonal coordinates just have
to "follow" these new axes.

<img src="projection.svg" alt="isometric projection" class="image-centered">

Here's the projection code in Rust

```rust
fn project((x, y, z): Voxel) -> (f64,f64) {
    // i and j are the coordinates in an orthogonal plane
    let i = f64::from(x - z);
    let j = f64::from(y - z);

    // transform the orthogonal plane into the isometric one
    let (dy, dx) =  f64::sin_cos(f64::to_radians(30.0));
    (i * dx - j * dx, i * dy + j * dy)
}
```


## Draw order

Draw each voxel from closest to farthest and making sure no triangles are drawn
twice. Note that the state of the edges doesn't matter because if the triangle
was already drawn it means there's a voxel that's closest to the eye that hides
whatever edges are in the farthest voxel.

To do so there's should be "nearness" value for each value that's bigger when
the voxel is closer to the eye. I'm quite sure there are quite a few different
approaches and formulas here, the one I went with is

```rust
pub fn nearness((x, y, z): Voxel) -> i64 {
    x + y + z
}
```

Another way to think about this function is simply the distance between the
origin and the voxel, the bigger the distance the closer the voxel. Note that
there's no need to do the correct calculation, but the Manhattan distance can be
used instead because voxels with the same nearness value end up being in
different positions in the isometric space.


## Conclusions

I had a lot of fun working on this project and I'm quite proud of it even though
it was really frustrating from times to times. Especially at the beginning when
I wasn't sure if what I was looking at was correct or just optical illusions.

Anyway, here are some images from various sketches that use this framework.
Until next time.

<img src="archi.png" alt="isometric architecture" class="image-centered">
<img src="octa.png" alt="octahedron" class="image-centered">

[ivo-github]: https://github.com/d-dorazio/r3d/#ivo-the-isometric-voxel-renderer
[wblut]: https://twitter.com/wblut?s=20&t=9dSF2Smqs3FbaDf0J9GgGQ
