---
layout: post
title:  "Building the Gator Compiler [1/4]"
categories: gator
---

## Overview

Gator is a graphics language developed by Cornell's [Capra group](https://capra.cs.cornell.edu/) (led by my advisor, [Adrian Sampson](https://www.cs.cornell.edu/~asampson/).  The goal of Gator is conceptually simple -- to extend graphics programming with *Geometry Types*, a novel type construction designed to reason about *geometry bugs* which can occur in graphics systems.  As argued in our paper [Geometry Types for Graphics Programming](https://dl.acm.org/doi/10.1145/3428241), published in OOPSLA 2020, these geometry bugs can cause subtle issues if not addressed properly.

The focus of this blog is not on the details of geometry needed to properly construct Geometry Types, however, but instead on the compiler we developed to allow reasoning about such types in a realistic setting.  In this four-part blog post, we will explore [the basics of geometry types](http://127.0.0.1:4000/~dgeisler/gator/2022/04/07/gator-compiler1.html), [some examples of the Gator language](http://127.0.0.1:4000/~dgeisler/gator/2022/04/07/gator-compiler2.html), [the actual compiler engineering](http://127.0.0.1:4000/~dgeisler/gator/2022/04/07/gator-compiler3.html), and finally [some potential (unexplored) extensions of this work](http://127.0.0.1:4000/~dgeisler/gator/2022/04/07/gator-compiler4.html).  As a whole, this post is meant to take a look at some of the engineering that went into building a semi-weird but also fairly simple research language.

It is perhaps worth noting that the details of the compiler described in the [third part](http://127.0.0.1:4000/~dgeisler/gator/2022/04/07/gator-compiler3.html) of this series go into some technical detail and assume that the reader has encountered or dealt with compilers before.  The rest of the blog is written to be read without understanding these details if you so prefer.  On the other hand, this blog is not written as a guide to using the Gator compiler, and is instead an overview of the challenges faced while constructing such a piece of software; in some sense, omitting the technical details loses some of the reason to read this post.  Reader beware?

### The Problem

Before diving into the details of constructing Gator, it is first necessary to understand the basics of geometry bugs and Geometry Types.  Geometry bugs are a class of bugs arising from confusing coordinate schemes (say, Cartesian or Polar coordinates), geometric objects (say vectors and points), and/or reference frames (say, the model or world frames, in graphics speak).  Geometry types, as previously mentioned, attempt to solve these problems by constructing a type with each of these three components.  A geometry type can be written with three components:

```
scheme<frame>.object
```

This syntax will hopefully be familiar to those with object-oriented language experience with one important caveat: both the parameterization *and* the class member of the coordinage scheme are part of the *type*.  This is a bit unusual, and takes some time to get used to -- however, we found this to be the most approachable representation after several syntax attempts.  To state the intuition explicitly: the coordinate scheme is parameterized with a reference frame, the combined elements of which form a class whose member is a geometric object.  With this construction, the geometric object can be thought of as providing the concrete representation of the type (e.g. a vector is the actual *thing* in Cartesian model space).

### Why Gator

Whenever building a compiler, the question start with is often: why not just build this type or idea into an existing language or toolchain?  In this context, the reasoning was simple: by building a compiler, we could reduce our iteration time on potentially complex type constructions and introduce novel type ideas without having to fiddle with implementing the desired syntax in an existing type system.  I believe this ended up being the right choice, as we were able to explore several representations of geometry types, both from a syntax and theory perspective, without having the extra work of making another system work.

This is not to say that Gator is the be-all end-all best choice for Geometry Types as a whole -- far from it.  In fact, I would go so far as to argue that implementing Geometry Types in C++, HLSL, [WGSL](https://www.w3.org/TR/WGSL/), a game engine like [Unity](https://unity.com/), or a more practical language like [Slang](http://graphics.cs.cmu.edu/projects/slang/) would be an excellent engineering step towards actually making these constructions usable in real graphics systems.  As a result, Gator is most likely to continue as a purely research language -- a starting point for others to use the ideas presented when engineering their own systems.

With the problem and justification for a compiler out of the way, let's take a dive into what kind of programs we'd like to be able to write, and commentary on some of the weird design of using Gator in Part 2.

part_1 [part_2](/~dgeisler/gator/2023/09/11/gator-compiler2.html) part_3 part_4