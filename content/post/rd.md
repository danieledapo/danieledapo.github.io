---
title: "Reaction Diffusion using the Gray-Scott model"
date: 2020-04-05T16:25:08+02:00
---

Recently I've been playing around with [Reaction Diffusion][reaction-diffusion]
mainly because I was intrigued by the patterns it was able to generate and because
a lot of cool people were doing the same.

I then started the dead-simple project [rd][rd]. I think I like short names.


## What even is Reaction Diffusion

Reaction Diffusion is a simulation of two chemicals reacting together where one
uses the other to grow and expand. Think of it like having a bowl where you
pour something into at a given rate and inside the bowl there's also another
substance that absorbes the first substance. The second substance also dies
"naturally" at a given rate.

The key parameters here are the rate at which the first chemical is fed into
the system (feed\_rate), the rate at which the second chemical is killed
(kill\_rate) and the initial state of the system before pouring the first
chemical into it. Changing any of these parameters greatly affects the final
results of simulation.

Now, the Wikipedia article on Reaction Diffusion was not very helpful to me
because it's way more "math-y" and complicated that what I can digest, but I
was lucky enough to find another very clear and [simple
explanation][reaction-diffusion-tutorial] of a model used to approximate this
process. It's a short read, but I suggest you to take a look at it even to just
look at the images.

Anyway, the whole simulation can be summarized in the following way:

- initialize the system to be a grid filled only with chemical A (the one
  that's poured into it). Note that it's not strictly necessary to use a grid,
  but it's quite easy to work with.
- pour some chemical B into the system by changing the quantity of chemical B
  of some cells so that the two chemicals can actually react and the system can
  evolve. On which cells chemical B is poured into can be completely random or
  driven by custom logic.
- now it comes the main loop which can be repeated as long as it's needed. The
  loop mainly consists of running a kernel convolution on each cell and
  updating the quantities of the two chemicals at each cell. I won't try to
  explain the exact formulas used in the simulation because I don't pretend
  I've fully understood them, but I can ensure they're not too hard to code.

Eventually, to render the system to an image I simply take the amount of
chemical B at each cell and convert to a grayscale pixel using a bit of
interpolation. Nothing too complicated.


## Rd

So, it didn't take me too much time to code something up to generate some
images representing the system and hacking something together to generate a
video from such frames. I'm not too proud of the overall design as I don't feel
like saving a bunch of images on disk just to call `ffmpeg` to then create a
video is a reasonable thing to do, but it works and it's good enough for now.

Rd can either start from a random seed state or take a grayscale image to
initialize the initial system. I like the former because the animation it
produces are quite lovely sometimes, but most often than not they're quite
boring. The latter is simply cool to watch.

Anyway, if you're interested take a look at the [repo][rd].


## Conclusions

I'm ok with the results and this new toy already proved quite entertaining for
quite some time. There are a lot of other things to try out, but I'm done with
Reaction Diffusion for now.

Anyway, I'd like to leave you with an animation of the simulation running with
my logo as the seed which I find particularly cool.

Stay home, I guess. See ya.

<video class="video-centered" muted autoplay loop>
  <source src="rd.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

[reaction-diffusion]: https://en.wikipedia.org/wiki/Reaction%E2%80%93diffusion_system
[rd]: https://github.com/d-dorazio/rd
[reaction-diffusion-tutorial]: https://www.karlsims.com/rd.html
