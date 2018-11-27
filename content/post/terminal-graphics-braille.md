---
title: "Terminal graphics with Braille characters"
date: 2018-11-27T21:21:53+01:00
draft: true
---

At work, I'm currently developing a 3D web CAD for modeling shoes which is
really interesting project, but also very challenging! Since the project is
quite complex by itself, we spend a lot of time writing tests and especially
making sure the code is [well documented][my-take-on-docs]. However,
documentation is mainly composed by text and sometimes a drawing is way more
useful than a lot of words even for simple concepts. For example, try to explain
what perpendicular and parallel mean without drawing anything. Chanches are you
already understand what they mean, but try to give a definition nonetheless.
It's quite hard.

Ok, so at this point we want to add drawings to the documentation and since
documentation is mainly text we have to resort to ASCII drawings! I really love
ASCII art, but I'm pretty bad at it. However, I really didn't want to include
images or other media in the documentation because it'd require me to switch to
a browser or something which I find really annoying. Being a lazy programmer, I
did what I do best: search for existing solutions and use those. Unfortunately,
there didn't seem to be any tools that would allow me to draw arbitrary shapes
using only text. I was surprised by this because I saw a lot of cool projects
that draw complex shapes on the terminal over the years. As I was googling more
I was getting convinced that drawing 3D geometries on the terminal is possible,
but that nobody enough was crazy enough to do it. Since I had a free weekend, I
decided to give it a shot.

## Braille characters

Initially, I tried to use the canonical `/`, `|`, `-`, `_` and `\`
characters to draw lines but that didn't work well mainly because I don't think
there's a way to draw a non vertical or horizintal line that doesn't look like a
staircase.

I did some research and eventually I stumbled upon the [`drawille`][drawille]
project which looked quite promising! By going through the `README.md` and the
examples I quickly noticed that the curves were rendered smoothly and that lines
didn't look as staircases too much. Also, among the examples in the `drawille`
repo there's a program that renders a 3D rotating cube. This was exactly what I
was looking for!

However, `drawille` is written in Python, but I planned to use Rust to build the
tool so I needed to understand how `drawille` worked in order to port it to
Rust. I took a look at the Python code and I realized how cool the baseline idea
is: leverage the dots in [Braille patterns][braille-patterns] to draw smooth
lines and curves.

In case you don't know, Braille is a tactile writing system used by blind people
where each symbol can contain at most 8 dots arranged in a 4 x 2 grid depending
on the symbol itself. We can then see each symbol as a collection of 8 pixels
that we can turn on and off independently and since each pixel is represented by
a circle drawing curves would not look too bad. Neat!

Clearly, the Unicode standard supports the whole Braille Patterns block which
contains 256 distinct symbols but it is also quite clever in how it defines the
codepoints. In fact, it maps each of the eight dots of a Braille symbol to one
bit in a byte which is the offset from the start of the Braille block `U+2800`.
This means that we can create the character corresponding to a given Braille
character by simply turning on the bits that map to a raised dot of a byte and
add that value to `U+2800`. It also means that if we have two different Braille
characters we can combine them into a character that has all the raised dots of
both characters by simply doing a bitwise or between the offsets.

In Python this idea can be summarized by the following snippet.

```python
def braille(off):
    return chr(0x2800 + off)

braille(0x1) # '⠁'
braille(0x2) # '⠂'
braille(0x1 | 0x2) # '⠃'
braille(0x1 | 0x2 | 0x04) # '⠇'
braille(0x1 | 0x2 | 0x04 | 0x08 | 0x10 | 0x20 | 0x40 | 0x80) # '⣿'
```

## Termesh

Ok, so back where we started. With this idea, I implemented a very simple canvas
that would basically turn on dots in a grid of Braille characters according to
which pixels I wanted. It has primitives to draw only lines and triangles, but
since the first almost quite working implementation I was quite satisfied and
honestly really surprised by the results!

As I was starting to have a more refined canvas implementation, I thought that
implementing a simple interactive STL renderer wouldn't be that difficult and
could give exciting results. And indeed they were. Somewhat. At this point I
didn't have any form of shading whatsoever so shapes looked strange to say the
least. However, once I implemented a dead simple flat shading algorithm I was
able to view 3D shapes in a decent way.

Here are a couple of screenshots rendering the Utah teapot.

![teapot2.png](https://raw.githubusercontent.com/d-dorazio/termesh/master/images/teapot2.png)
![teapot3.png](https://raw.githubusercontent.com/d-dorazio/termesh/master/images/teapot3.png)

They're far from being perfect, but I can live with those.

The only problem that bothers me a bit but that I don't know how to solve is
that I cannot set the color of each pixel separately because I can only change
the color on a by character granularity, but each character contains more
pixels!

## DSL

At this point the main issue was solved because I was able to draw somewhat
pleasing 3D shapes using only text. I could then copy and paste the render to
the documentation. However I was still a bit unsatisfied because I still had to
manually create an STL to then render to text. Therefore I decided to write a
very simple domain specific language that I could use to describe arbitrary
shapes. Also, a bonus point of having a DSL is that I can version control the
code easily because it's not a binary format, but just text!

Here's an example of the language.

```python
# define vertices
vertex tip  = 1 1 2
vertex b1   = 0 0 0
vertex b2   = 0 2 0
vertex b3   = 2 2 0
vertex b4   = 2 0 0

# build triangles
triangle tip b1 b2
triangle tip b2 b3
triangle tip b3 b4
triangle tip b4 b1

# build lines
vertex v0 = -0.5 -0.5 0
vertex v1 =  2.5  2.5 2
line v0 v1
```

As you can see, the language is really minimal as of now, but it's already
possible to build relatively complex objects. However, I plan to add more
features to the language to make it more ergonic to use (namely I'd like to
support arithmetic operations at least).

## Conclusion

I find quite funny that symbols made for blind people can make non-blind people
see more. Now that I think about it, it's probably more philosophical than
funny, but that's not my thing.

Anyway, if you'd like to give a look at the code you can checkout the [`termesh`
repo][termesh-repo].

[my-take-on-docs]: {{< relref "my-take-on-documenting-code.md" >}}
[drawille]: https://github.com/asciimoo/drawille
[braille-patters]: https://en.wikipedia.org/wiki/Braille_Patterns
[termesh-repo]: https://github.com/d-dorazio/termesh
