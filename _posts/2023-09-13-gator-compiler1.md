---
layout: post
title:  "Building the Gator Compiler [1/4]"
categories: gator
---

## Overview

Gator is a graphics language developed by Cornell's [Capra group](https://capra.cs.cornell.edu/) (led by my advisor, [Adrian Sampson](https://www.cs.cornell.edu/~asampson/).  The goal of Gator is conceptually simple -- to extend graphics programming with *geometry types*, a novel type construction designed to reason about *geometry bugs* which can occur in graphics systems.  As argued in our paper [Geometry Types for Graphics Programming](https://dl.acm.org/doi/10.1145/3428241), published in OOPSLA 2020, these geometry bugs can cause subtle issues if not addressed properly.

The focus of this blog is not on the details of geometry needed to properly construct Geometry Types, however, but instead on the compiler we developed to allow reasoning about such types in a realistic setting.  In this four-part blog post, we will explore the basics of geometry types, some examples of the Gator language, what pieces make up the Gator compiler, and finally some stories on the challenges in building this compiler.  As a whole, this post is meant to take a look at some of the engineering that went into building a semi-weird but also fairly simple research language.

It is perhaps worth noting that the details of the compiler described in the third part of this series go into some technical detail and assume that the reader has encountered or dealt with compilers before.  The first two sections make no such assumptions, and only expect that the reader has some experience with programming.

### The Problem

Before diving into the details of constructing Gator, it is first necessary to understand the basics of geometry types.  Geometry types are meant to help with a class of bugs arising from confusing coordinate schemes (say, Cartesian or Polar coordinates), geometric objects (say vectors and points), and/or reference frames (say, the model or world frames, in graphics speak).  We attempt to solve these problems by constructing a type with each of these three components:

```
scheme<frame>.object
```

This syntax will hopefully be familiar to those with object-oriented language experience with one important caveat: both the parameterization *and* the class member of the coordinage scheme are part of the *type*.  This is a bit unusual, and takes some time to get used to -- however, we found this to be the most approachable representation after several syntax attempts.  To state the intuition explicitly: the coordinate scheme is parameterized with a reference frame, the combined elements of which form a class whose member is a geometric object.

Some examples of geometry types are:

```
cart3<world>.point
polar<model>.vector
cart2<world>.transformation<clip>
```

To quickly summarize these types and the intuition of the data structure that might match the types:

```
// 3-dimensional cartesian point in the world frame
// probably represented by a 3-tuple or array
cart3<world>.point

// polar vector in the model frame
// polar coordinates are 2-dimensional, so probably represented by a 2-tuple
polar<model>.vector

// 2-dimensional cartesian transformation in the world frame to the clip frame
// this can be read as a map from world to clip
cart2<world>.transformation<clip>
```

If you aren't familiar with `model`, `world`, or `clip` frames, they are not necessary to understand for this article (if you'd like to read more, I'd recommend the [OpenGL introduction to coordinate systems](https://learnopengl.com/Getting-started/Coordinate-Systems)).  Similarly, no need to recall exactly how cartesian and polar coordinates work, though it is important to note that the dimension of a coordinate scheme does matter (2-dimensional has two pieces of information, such as `x` and `y` while 3-dimensional adds another).  I will speak more about the geometric objects `point`, `vector`, and `transformation` in the next part of this post.

Why we believe this syntax to be a reasonable choice somewhat off-topic from this post; if you're interested in exploring this in excessive detail, consider reading [our paper](https://dl.acm.org/doi/10.1145/3428241) instead.  What I will instead be focusing on in this post is describing what we needed to do to make this sort of syntax a reality (and what steps were useful for exploring choice of syntax and programs as a whole).

### Why Gator

When building a research compiler, the question start with is often: why not just build this type or idea into an existing language or toolchain?  In this context, the reasoning was simple: by building a compiler, we could reduce our iteration time on potentially complex type constructions and introduce novel type ideas without having to fiddle with implementing the desired syntax in an existing type system.  I believe this ended up being the right choice, as we were able to explore several representations of geometry types, both from a syntax and theory perspective, without having the extra work of making another system work.

This is not to say that Gator is the be-all end-all best choice for implementing geometry types for real use -- far from it.  In fact, I would go so far as to argue that implementing geometry in C++, HLSL, [WGSL](https://www.w3.org/TR/WGSL/), a game engine like [Unity](https://unity.com/), or a more practical language like [Slang](http://graphics.cs.cmu.edu/projects/slang/) would be an excellent engineering step towards actually making these constructions usable in real graphics systems.  As a result, Gator is most likely to continue as a purely research language -- a starting point for others to use the ideas presented when engineering their own systems.

With the problem and justification for a compiler out of the way, let's take a dive into what kind of programs we'd like to be able to write, and commentary on some of the weird design of using Gator in Part 2.

part_1 part_2 part_3 part_4