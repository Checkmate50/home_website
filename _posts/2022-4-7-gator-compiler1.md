---
layout: post
title:  "Building the Gator Compiler [1/3]"
date:   2022-4-07 11:52:28 -0400
categories: gator
---

## Overview

Gator is a graphics language developed by Cornell's [Capra group](...) (led by my advisor, [Adrian Sampson](...).  The goal of Gator is conceptually simple -- to extend graphics programming with *Geometry Types*, a novel type construction designed to reason about *geometry bugs* which can occur in graphics systems.  As argued in our paper [Geometry Types for Graphics Programming](), published in OOPSLA 2020, these geometry bugs can cause subtle issues if not addressed properly.

The focus of this blog is not on the details of geometry needed to properly construct Geometry Types, however, but instead on the compiler we developed to allow reasoning about such types in a realistic setting.  In this four-part blog post, we will explore [the basics of geometry types](...), [some examples of the Gator language](...), [the actual compiler engineering](...), and finally [some potential (unexplored) extensions of this work](...).  As a whole, this post is meant to supplement our paper (though reading the paper is not a prerequisite for understanding this blog series).

It is perhaps worth noting that the details of the compiler described in the [third part](...) of this series are fairly technical and expect a reasonable understanding of the basics of compiler design.  The rest of the blog is written to be read without this section if you so prefer.  On the other hand, this blog is not written as a guide to using the Gator compiler, and is instead an overview of the challenges faced while constructing such a piece of software.

### The Problem

Before diving into the details of constructing Gator, it is first necessary to understand the basics of geometry bugs and Geometry Types.  Geometry bugs are a class of bugs arising from confusing coordinate schemes (say, Cartesian or Polar coordinates), geometric objects (say vectors and points), and/or reference frames (say, the model or world frames, in graphics speak).  Geometry types, as previously mentioned, attempt to solve these problems by constructing a type with each of these three components.  A geometry type can be written with three components:

```
scheme<frame>.object
```

This syntax will hopefully be familiar to those with object-oriented language experience with one important caveat: both the parameterization *and* the class member of the coordinage scheme are part of the *type*.  This is a bit unusual, and takes some time to get used to -- however, we found this to be the most approachable representation after several syntax attempts.  To state the intuition explicitly: the coordinate scheme is parameterized with a reference frame, the combined elements of which form a class whose member is a geometric object.  With this construction, the geometric object can be thought of as providing the concrete representation of the type (e.g. a vector is the actual *thing* in Cartesian model space).

### Why Gator

Whenever building a compiler, it is important to ask yourself: why not just build this type or idea into an existing language?  For us, the reasoning was simple: by building a compiler, we could reduce our iteration time on potentially complex type constructions and introduce novel type ideas without having to fiddle with implementing the desired syntax in an existing type system.  This choice proved invaluable as we explored several representations of geometry types, both from a syntax and theory perspective.

This is not to say that Gator is the be-all end-all best choice for Geometry Types as a whole -- far from it.  In fact, I would go so far as to argue that implementing Geometry Types in C++, HLSL, or [Slang](...) (another research graphics language) would be an excellent engineering step towards actually making these constructions usable in real graphics systems.  As a result, Gator is most likely to continue as a research language -- a starting point for others to use the ideas presented when engineering their own systems.

---

## Examples

While some example snippets of Gator are provided in [our paper](...), 

### Geometry Types in Action

### Standard Library

---

## Compiler Structure

### Gator Compilation

## Engineering the Compiler

### Lexing/Parsing in Ocaml

### AST Structure

### Contexts

### Functions and Dispatch

---

## Interesting Extensions

### TypeScript Compiler

### Unit Types

### Formalizing Algebraic Geometry

## Conclusion

### Should you use Geometry Types?