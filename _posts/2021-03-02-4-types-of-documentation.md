---
layout: post
title: "4 types of documentation"
date: 2021-03-02 00:00:00 +0000
---

The other day I was listening [The art of reading the docs episode](https://changelog.com/gotime/167) from the Go Time podcast and someone mentioned this [What nobody tells you about documentation talk](https://www.youtube.com/watch?v=t4vKPhjcMZg) from Daniele Procida at PyCon AU 2017.

Daniele argues there are 4 types of documentation, each of one serving a different purpose:

- Discussions
- Reference
- How-to guides
- Tutorials

## Tutorial

Tutorials are learning-oriented. A good tutorial is a series of steps that help the reader to learn something by doing.

Daniele goes further suggesting learning is not the most important element but providing the reader an enjoyable experience. When that happens, the reader will want to do it again in the future and learning becomes a by-product of the experience (I wished my school teachers thought that way back in time).

Some characteristics of tutorials are:

- deal with concrete (not abstractions).
- lean by doing,
- sense of achievement,
- repeatability,
- how to get started,

The outcome of a tutorial is to help someone to do something and at this stage it is OK to not fully understand what's going on.

## How-to guide

How-to guides are problem-oriented. It generally contains a sequence of steps that, if followed in order, guide the reader through solving a specific issue.

As tutorials are commonly used by readers at the beginning of their learning journey, they might not have enough knowledge to come up with meaningful questions yet. Once they acquire a better understanding and uncover specific issues, a how-to guide is what they need.

A recipe is an example of a how-go guide as it has a clear goal and describes a sequence of steps to achieve that goal.

Some properties of how-go guides are:

- practical usability.
- sequence of steps,
- focused on a specific issue,

## Reference

Reference guides are descriptions of something (like a library, framework or programme) and how to use it. It's information-oriented.

Hopefully when someone get to the point of reading a reference guide they have a good understanding of the mechanics of the thing in question and how to perform some tasks with it.

Some common properties of reference guides are:

- structure.
- description,
- to the point,

[Go docs](https://golang.org/pkg/) and programming language documentation in general are a good examples of this.

## Discussion

Discussion or explanation are understanding-oriented and offer clarifying background and context on a topic. Books, conference talks, essays are some examples.

Good discussions focus on:

- making connections with other topics, concepts.
- explaining the "why",
- providing context,

## Effective documentation

With these 4 types of documentation, it can be challenging for authors not to mix them together as sometimes boundaries can blur. This leads to documentation debt, which makes things tedious, time consuming and ineffective.

It is often the case different types of documentation need one another, like How-to guides linking to References, or Tutorials referring to Discussions for further background. The key though is stepping across boundaries should be optional, allowing readers to either focus on what they want to achieve or taking a detour.

To go further on this check [this guide](https://documentation.divio.com/) written by Daniele.
