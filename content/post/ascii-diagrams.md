---
title: "Ascii Diagrams"
date: 2020-02-21T21:13:37+01:00
---

The 25th challenge of [last year AoC](https://adventofcode.com/2019/) was
particularly funny to solve because it didn't require a lot of programming, but
a bit of play and intuition. I don't want to reveal too much about the
challenge, but it's enough to say that drawing a chart or diagrams is quite
useful.

At the time I used [Graphviz](https://www.graphviz.org/) without thinking too
much about it as it's a great piece of software and it worked great. However, I
wanted the final diagram to be ASCII so that I could embed it in source files.
As of now, graphviz has no such backend.

Now, a sane person might have looked into how to write such backend, but
instead I tried to create an ASCII diagram renderer from scratch. I created
[ascii-diagrams](github.com/danieledapo/ascii-diagrams).

## Problem

I decided to start very small so I decided to render blocks only at fixed
positions while trying to find the best edge arrangement to connect such blocks
automatically.

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

- start from the current best solution,
- select an edge randomly,
- select two points randomly on the blocks that are connected by the selected
  edge,
- find the shortest path between those two points,
- if the overall diagram is "simpler" than the current solution then keep the
  new solution, discard it otherwise.

This process is repeated a bunch of times until no intersections are found or
up to a given number of times.

This simple process is actually able to give decent results, but all the magic
lies in how the initial solution is built.

For example, edges between adjacent blocks are always placed in the most
obvious way (just straight lines) because that's almost always the correct
layout. These edges are then excluded from the main tweaking algorithm as
they're hardly wrong.

The remaining edges are sorted by the logical distance between the blocks and
then placed in the best way. The reason the edges are processed according to
their "logical" length is that short edges are easier to get right, but more
importantly we don't want to get them too wrong because shorter edges should be
visually simpler than longer ones to make the overall diagram easier to read.

## Conclusions

This was a short post and probably not that exciting, I know. To be frank I
didn't really feel like writing (or publishing) it, but given that this year
resolution is to publish a post per month and that I'm already one month late I
figured I should just write it anyway.

I'll end this post with a couple of diagrams `ascii-diagrams` is able to
generate. Take a look at the
[repo](https://github.com/danieledapo/ascii-diagrams) if you'd like to know more
about how to actually use it.

![diag3](diag3.png)
