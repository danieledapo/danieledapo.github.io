---
title: "Buzz, the ray tracer!"
date: 2019-06-23T22:29:14+02:00
draft: true
---

Recently I've been working on a toy ray tracer to learn a bit about how 3D
renderers work. I also took the opportunity to learn a bit of math and a whole
ton about computer graphics. This post isn't a tutorial on how to build a ray
tracer because there's a ton of resources online already, but it is mostly the
journey I went through writing Buzz.

For the impatient, the source code of the whole project lives on
[github][r3d-repo].

## How it all started

More than a month ago I was watching Toy Story (again) which is a thing I always
love to do from time to time, but that time I was particularly attracted by a
scene where Buzz Light-Year uses its laser to shoot rays everywhere. That scene
reminded me of a ray tracer and so I decided I should build one.

I don't know very much about 3D and ray tracing was no exception, so the first
thing I had to do was to gather some resources. To my surprise, I found out that
there is a lot of documentation and tutorials on ray tracing! After a bit of
searching, I decided to follow [Writing a Ray Tracer in a Weekend][wrtiw-repo]
because it seemed a good compromise between theory and practice.

## Ray Tracing 101

A ray tracer is a type of 3D renderer whose only purpose is to transform a 3D
scene in an image.

The core idea behind ray tracing is to simulate the way light behaves in the
real world to achieve the most accurate rendering possible. Clearly, the
downside is that is more resource intensive than other rendering methods like
[rasterisation][rasterisation] but the rendered images are more realistic.

The basic algorithm works by shooting lights rays from the camera into the scene
passing through each pixel of the output image. Then calculate the intersection
with the objects in the scene. If there is indeed an intersection then calculate
how the light refracts and reflects with that object and recursively shoot those
rays into the scene. On the other hand, if no intersection is found then it
means that the ray hits the environment. In pseudo-code it will be something
like

```python
for (x, y, pixel) in output_image:
    for _ in 0..nsamples:
        color = shoot_ray(ray_going_through(camera, (x, y)))
        pixel = blend(pixel, color)
```

Obviously I've omitted where all the magic happens (in `shoot_ray`) but I think
that is a good summary of how ray tracing works.

## Materials

<!--
  TODO: explain that lights change according to the material of the object that
  it hits. Mention the Lambertian, Dielectric and Metallic materials which are
  the ones Buzz support. This is the hard part because ray tracing is really all
  about modeling how the light behaves with materials. Mention PBR materials.
-->

## CSG

<!--
  TODO: explaing what CSG is and why is interesting.
-->

## Conclusion

<!--
  TODO: Show some renderings and overall conclusions.
-->

[r3d-repo]: https://github.com/d-dorazio/r3d.git
[wrtiw-repo]: https://github.com/petershirley/raytracinginoneweekend
[rasterisation]: https://en.wikipedia.org/wiki/Rasterisation
