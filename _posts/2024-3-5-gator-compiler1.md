---
layout: post
title:  "Building the Gator Compiler [1/3]"
categories: gator
hidden: true
---

## Overview

Gator is a graphics language developed by Cornell's [Capra group](https://capra.cs.cornell.edu/) (led by my advisor, [Adrian Sampson](https://www.cs.cornell.edu/~asampson/)).  The goal of Gator is conceptually simple -- to extend graphics programming with *geometry types*, a novel type construction designed to reason about *geometry bugs* which can occur in graphics systems.  As argued in our paper [Geometry Types for Graphics Programming](https://dl.acm.org/doi/10.1145/3428241), published in OOPSLA 2020, these geometry bugs can cause subtle issues if not addressed properly.

The focus of this blog is not on the details of geometry needed to properly construct Geometry Types, however, but instead on the compiler we developed to allow reasoning about such types in a realistic setting.  In this three-part blog post, we will explore the basics of geometry types, some examples of the Gator language, what pieces make up the Gator compiler, and finally some stories on the challenges in building this compiler.  As a whole, this post is meant to take a look at some of the engineering that went into building a semi-weird but also fairly simple research language.

It is perhaps worth noting that the details of the compiler described in the second part of this series go into some technical detail and assume that the reader has encountered or dealt with compilers before.  The first and third parts make no such assumptions, and only expect that the reader has some experience with programming.

### Why Gator

When building a research compiler, the question start with is often: why not just build this type or idea into an existing language or toolchain?  In this context, the reasoning was simple: by building a compiler, we could reduce our iteration time on potentially complex type constructions and introduce novel type ideas without having to fiddle with implementing the desired syntax in an existing type system.  I believe this ended up being the right choice, as we were able to explore several representations of geometry types, both from a syntax and theory perspective, without having the extra work of making another system work.

This is not to say that Gator is the be-all end-all best choice for implementing geometry types for real use -- far from it.  In fact, I would go so far as to argue that implementing geometry in C++, HLSL, [WGSL](https://www.w3.org/TR/WGSL/), a game engine like [Unity](https://unity.com/), or a more practical language like [Slang](http://graphics.cs.cmu.edu/projects/slang/) would be an excellent engineering step towards actually making these constructions usable in real graphics systems.  As a result, Gator is most likely to continue as a purely research language -- a starting point for others to use the ideas presented when engineering their own systems.

With the problem and justification for a compiler out of the way, let's look at an overview of what kind of programs we'd like to be able to write.  We will not be examining why these decisions were made in this post -- the motivation of why design a language in this way is the focus of our paper and months of discussion on our goals, and separate from implementing these goals.

### The Problem

Before diving into the details of constructing Gator, it is first necessary to sketch the problem we would like to solve.  To avoid going into the geometry problems Gator purports to solve, we will instead focus on the literal syntax and semantics we would like to support.  If you are interested in the problem being solved here, I would encourage reading our paper instead, where the details here are discussed in great detail.

At a basic level, Gator needs to be able to support types of the following form:
```
coordinate<frame>.object
```
`coordinate`, `frame`, and `object` are all strings, which I have given a name purely so we can unambiguously refer to each.  When we have all three of these pieces, we call such a type a "geometry type".  We would like to be able to give variables geometry types, in such a way that each component is distinct:
```
coordinate1<frame1>.object1 x = ...;
coordinate1<frame1>.object2 y = ...;
coordinate2<frame1>.object1 z = ...;
...
```
The key feature of Gator is typechecking operations involving variables of these types, specifically arithmetic operations and function calls.  Given how long a geometry type can be to write, we will also want simple type deduction with `auto`:
```
coordinate1<frame1>.object1 x = ...;
coordinate1<frame2>.object1 y = foo(-x);

float s = ...;          // note that we still have "regular" types like float or bool
auto z = s * (x + y);   // z will have a geometry type here, as inferred by `auto`
```

#### Custom Types

We further want to allow the user to introduce strings that can be used by the typechecker:
```
type scalar is float;           // we can have a more specific float called "scalar"

with frame(2) r:                // we can only use frames of dimension 2 for these types
coordinate cart2 : geometry {   // builds a new 'coordinate' we can use (ignore 'geometry' for now)
    object vector is float[2];  // builds a new 'object' we can use
    ...
}

frame screen has dimension 2;   // declares a new 'frame' of dimension 2
frame model has dimension 3;    // declares a new 'frame' of dimension 3
```
Observe that the declaration of `coordinate` is a bit more complex than that for `frame` or `object`.  The reason for this takes the intuition of using `coordinate<frame>` syntax -- we are _parameterizing_ the `coordinate` part of the type with our given `frame`.  We additionally have the restriction that each coordinate can only be parameterized with frames of the given (constant) dimension.  In our example above, we could then parameterize `cart2<screen>` but not `cart2<model>`.

#### Operations

Finally, the last piece we need for introducing Gator is the ability to actually say how these types interact with the arithmetic operations we would like to typecheck.  A core feature of Gator is the ability to overload any arithmetic operation with fairly complicated type information, so let's take a look at what a declaration here might look like:
```
// the same declaration as above
with frame(2) r: 
coordinate cart2 : geometry {
    object vector is float[2];
    
    // let's expand the `...` a bit

    // addition between vectors
    vector +(vector x, vector y) {                      
        // we need to force the types here, we use `as!` for type coercion
        // note that we are using `+` to be overloaded on vector-vector addition
        return (x as! vec2 + y as! vec2) as! vector;   
    }

    // subtraction between vectors is nearly identical
    vector -(vector x, vector y) {                      
        return (x as! vec2 - y as! vec2) as! vector;
    }

    // we also want to be able to multiply with a float
    // we need to have a generic float `T`, since a float can be made more specific
    // for example, we could use both a `float` and a `scalar` here
    with float T: vector *(vector v, T s) {
        return (v as! vec2 * s) as! vector;
    }
}
```

#### Example

We will conclude this section with an example of a valid program we can write given these declarations above.  Note that I will be relying entirely on intuition for this being a type-valid program for now -- Part 3 will go into much more detail on the type rules we would like to maintain:
```
// showing the frame declarations again for clarity
// the coordinate and object declarations above are assumed
frame screen has dimension 2;
frame proj has dimension 2;

cart2<screen>.vector v1 = ...;
cart2<screen>.vector v2 = ...;
cart2<proj>.vector v3 = ...;
scalar s = ...;

// this should be valid
// we could also just use `auto` for the type of `result`
cart2<screen>.vector result = s * (v1 + v2) - v2;

// note that the following code would be _invalid_ per our restrictions:
// intuitively, this is due to `proj` not being literally the same as `screen`
// auto bad = v1 + v3;
```

### Next Steps

Now that we have established what we would like this language to look like, we will take a look at the construction of the Gator compiler, and the pieces needed to support this language.  For this look at the pieces of the Gator compiler, we will examine all the structure _except_ typechecking, where we will go into more detail (including some extra typing information) in Part 3.

part_1 part_2 part_3