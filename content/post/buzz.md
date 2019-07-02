---
title: "Buzz, the ray tracer!"
date: 2019-06-23T22:29:14+02:00
draft: true
---

Recently I've been working on a toy ray tracer to learn a bit about how 3D
renderers work. I also took the opportunity to learn a little bit of math and a
whole ton about computer graphics in general. This post isn't a tutorial on how
to build a ray tracer because there's a ton of resources online already; it is
mostly the journey I went through writing Buzz.

For the impatient, the source code of the whole project lives on
[github][r3d-repo].


## How it all started

More than a month ago I was watching Toy Story which is a thing I always love
to do from time to time. That time I was particularly attracted by a scene
where Buzz LightYear uses its laser to shoot rays everywhere to locate the
enemies. That scene reminded me of how ray tracers work and so I decided to
build one.

I don't know very much about 3D and sure enough ray tracing was no exception,
so the first thing I had to do was to gather some resources and read some
documentation. To my surprise, I found out that there is a lot of documentation
and tutorials on how ray tracers work! After a bit of searching, I decided to
follow [Writing a Ray Tracer in a Weekend][wrtiw-repo] because it seemed a good
compromise between theory and practice. I don't regret this choice, but
probably it was a bit too light on theory.

## Ray Tracing 101

A ray tracer is a type of 3D renderer whose only purpose is to transform a 3D
scene into an image in the most accurate way possible. In order to do that it
shoots a _bunch_ of light rays into the scene and it then simulates how such
rays behave to find out the pixels of the final image.

If this sounds expensive it really is way more expensive than
[rasterisation][rasterisation] for example, but on the other hand the rendered
image is way more accurate especially when looking at shadows.

The core idea of the algorithm can be summarized by the following code snippet.

```python
for (x, y) in output_image:
    for _ in 0..nsamples:
        color = shoot_ray(ray_going_through(camera, (x, y)))
        output_image.set_pixel(x, y, color)
```

Quite neat, isn't it? Obviously I've omitted where all the magic happens (that
is in `shoot_ray`) but I think that is a good overview of how ray tracing
works. It basically goes through each pixel of the final image, shoots slightly
different rays into the scene and eventually blends all the potentially
different colors for the same pixel.

The different part of ray tracing is in writing a good `shoot_ray` function.
Conceptually, all it does is to find the closest intersection, if any, between
the ray and the scene and then according to which object it hit and its
material it returns a plausible color.


## Materials

Coming up with a faithful representation of materials is hard and it turns out
that this is very much a research area. In fact there are a lot of ways to
model materials with varying degrees of complexity. I choose to implement only
the lambertian, metal and dielectric materials because they seemed the most
basic.

Among the three, the lambertian material was the easiest to implement because it
turns out that a relatively good model consists in simply bouncing the ray along
the normal of the intersected object.

Metal materials were a bit harder to implement because they do not simply make
the incoming ray bounce in a random direction but instead such ray is reflected
and then slightly perturbed. Intuitively this makes sense because pure metals
simply reflect light rays.

Dielectric materials were the hardest to understand and to code. These materials
look like glass and do not simply reflect incoming rays but instead they let
some amount of rays go through the object (refraction) and some others are
reflected. The ratio between reflected and refracted rays are ruled by the
[Fresnel factor][fresnel-factor] which I decided to simulate by using [Schlick's
approximation equations][Schlick-approx]. Practically this means that each time
a ray hits a dielectric material the ray tracer randomly generates a probability
and according to whether it's greater or smaller than the Fresnel factor for
that particular material it reflects or refracts the incoming ray.

<!-- TODO: image with examples of these materials -->

Note that these are very simple materials and do not look realistic because real
materials but that's fine to me.


## Constructive Solid Geometry

While studying ray tracers and in general learning more about 3D rendering I
came across the concept of [Constructive Solid Geometry][csg] or CSG. The core
idea is to have simple primitive objects like spheres, cubes, etc... and then to
use boolean operators (i.e. union, intersection and difference) to combine them
together. Being a big fan of functional programming and composition in general,
this concept really spoke to me and I decided that buzz should be able to render
CSG shapes.

The implementation was quite straightforward, I modelled the primitives using
[Signed Distance Functions][signed-distance-functions] or SDF. Signed distance
functions are a way to look at objects as functions that return the distance
from a point to the object's surface. If the distance is negative than the point
is inside the object, if it's zero then it's exactly on the surface and if it's
positive it's outside the object. Here's an example SDF for spheres

```python
class Sphere:
    center: Vec3
    radius: float

    def dist(self, point):
        return (point - self.center).norm() - self.radius

```

Implementing the boolean operators on these primitives took me a bit of time,
but I enjoyed the process nonetheless. The union between two SDF is simply the
minimum distance between the two while the intersection is simply taking the
maximum. Intuitively this means that the distance of a union object to a point
is the distance of the closest object to that point and symmetrically for the
intersection object. The difference operator is a bit more involved but it boils
down to taking the intersection between the first object and everything but the
second that is the intersection between the first object and the negation of the
second. The negation can be done by simply changing the sign of a SDF.

```python
def union(sdf1, sdf2, point):
    return min(sdf1.dist(point), sdf2.dist(point))

def intersection(sdf1, sdf2, point):
    return max(sdf1.dist(point), sdf2.dist(point))

def difference(sdf1, sdf2, point):
    return max(sdf1.dist(point), -sdf2.dist(point))
```

This is all pretty cool but unfortunately this isn't enough to render CSG
objects because usually ray tracers need to find the intersection point on the
object to properly calculate the normal for shading, etc... However, CSG only
can only provide distances because that's the whole point of representing models
using CSG after all. Fear not though, there's a technique known as ray marching
that basically consists in moving along the incoming ray by the distance from
the SDF object to the ray origin until such distance is 0 that is the
intersection point.

<!-- TODO: image -->


## Conclusion

Ray tracers are pretty cool and are pretty interesting to implement, but it's
not a walk in the park if you don't know much about 3d math. The best part of
this journey was that I could see the results of my code changes pretty much
immediately.

I might try to implement [Physically based rendering][pbr] at some point in the
future, but I'm done with ray tracing for the foreseeable future. The next thing
is to build a rasterizer to explore the other face of 3D rendering. See you
then.

<!-- TODO: Show some images -->

[r3d-repo]: https://github.com/d-dorazio/r3d.git
[wrtiw-repo]: https://github.com/petershirley/raytracinginoneweekend
[rasterisation]: https://en.wikipedia.org/wiki/Rasterisation
[fresnel-factor]: https://en.wikipedia.org/wiki/Fresnel_equations
[Schlick-approx]: https://en.wikipedia.org/wiki/Schlick's_approximation
[pbr]: https://en.wikipedia.org/wiki/Physically_based_rendering
[csg]: https://en.wikipedia.org/wiki/Constructive_solid_geometry
[signed-distance-functions]: https://en.wikipedia.org/wiki/Signed_distance_function
