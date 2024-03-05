---
layout: post
title:  "Church Encodings"
categories: church
hidden: true
---

## A Concrete Look at Lambda Calculus

Lambda calculus can seem to be a theoretical (and perhaps intimidating) construction, only useful for academic study and confusing hapless students.  In this post, I would like to explore a much more direct look at doing computation with the lambda calculus.

Specifically, let's take a look at computation using [JavaScript arrow expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions).  You can do similar computation in most any language with anonymous functions, but JavaScript is both fairly lightweight and has fairly easy to read anonymous function syntax.  Additionally, JavaScript is easy to run in the browser, so I can more easily provide example code that you can just copy-paste to see how it works.

I will assume no familiarity with the formal ideas of the lambda calculus.  I will, however, assume that the reader is somewhat fimiliar with the concept of an [anonymous function](https://en.wikipedia.org/wiki/Anonymous_function).

### Lambda Calculus Basics

