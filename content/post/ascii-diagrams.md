---
title: "Ascii Diagrams"
date: 2020-01-28T19:13:37+01:00
draft: true
---

The 25th challenge of [last year Aoc](https://adventofcode.com/2019/) was
particularly funny to solve because it didn't require a lot of programming, but
a bit of play and intuition. I don't want to reveal too much about the
challenge, but it's enough to say that drawing a chart or diagrams is quite
useful.

At the time I used [Graphviz](https://www.graphviz.org/) without thinking too
much about it as it's a great piece of software and it worked great. However, I
wanted the final diagram to be ASCII so that I could embed it in source files.
As of now, graphviz has no such backend.

Now, a sane person might have looked into how to write such backend, but
instead I tried to create an ASCII diagram renderer from scratch. I started
then [ascii-diagrams](github.com/d-dorazio/ascii-diagrams).

This is an example of a diagram rendered using `ascii-diagrams`.

TODO: image


## Problem

I decided to start very small so I decided to render blocks only at fixed
positions while trying to find the best edge arrangement to connect such
blocks automatically.

Given that positions are already known, rendering blocks was not particularly
hard. I had to pay a bit of attention to handle padding and margins properly,
but nothing too complicated.

On the other hand, finding a good arrangement of lines is more complicated
because a lot of factors have to be taken into account and fundamentally it's a
NP problem.

Since the search space is so huge, it's practically impossible to visit all the
possible solutions to find the best one. We have to resort to some form of
dynamic programming to find a solution "good enough" for our use cases.

The strategy I used eventually boils down to:
- start from the current best solution
- select an edge randomly
- select two points randomly on the blocks that are connected by the selected
  edge,
- find the shortest path between those two points
- if the overall diagram is simpler than the current solution then keep the new
  solution, discard it otherwise

This process is repeated a bunch of times until no intersections are found or
up to a number of times.

This simple process is actually able to give decent results, but all the magic
lies in how the initial solution is built.


## Initial solution


## Easy algorithm


## Conclusions

As simple as it might sound, it does give some pretty good results like the following.
