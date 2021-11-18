---
layout: post
title:  "An Introduction to Mutation Testing [1/3]"
date:   2021-11-01 11:52:28 -0400
categories: mutation testing
---

# Send to recursive_error and Lunar

# Overview

This article looks to give the motivation for and theory behind mutation testing.  In particular, I will be discussing how mutation testing works, some theory behind why mutation testing works, some problems with mutation testing (and when to consider using it), and a short selection of applications at the time of this writing.  The intended audience is anyone familiar with some programming ideas, some knowledge of testing, but is written at a level you should need no prior knowledge of mutation testing or related approaches.

Further or in-depth reading recommendations, along with the main citations for this article:\\
[An Analysis and Survey of the Development of Mutation Testing](https://ieeexplore.ieee.org/document/5487526)\\
[Chapter Six - Mutation Testing Advances: An Analysis and Survey](https://www.sciencedirect.com/science/article/abs/pii/S0065245818300305)\\
[A list of some major mutation testing projects](https://awesomeopensource.com/projects/mutation-testing)

# A Short Thought Experiment

Consider some very simple C++ code with a (non-obvious) problem:

{% highlight cpp %}
int foo(int x) {
  return x + 1;
}

void check() {
  int x = foo(20);
  assert(x > 0);
}
{% endhighlight %}

Clearly, this code _works_, and the assertion passes; however, this assertion isn't as tight as it could be.  In particular, we assert that `x > 0` and not the "tighter" `x > 20`.  This program is thus **under-specified**, meaning that there could be a stronger bound on the assertion (or test, in some sense) that we've written.

This tightening is important for a variety of reasons that we'll explore later, but this example code is hopefully immediately unsatisfying.  If we can have a tighter assertion, should we not have one?  If we can have a better description of our code (in the form of a specification of this function), shouldn't we have such a tighter description?

Since this test reports correct code though, how in the world should we go about actually finding that the assertion is not as tight as it can be?  Finding under-specified functions is very hard -- we need some way to identify cases where the code is working but is prone to "still working" even when things change.  In particular, we need some way to detect when code is prone to "instability", that is, still working when it shouldn't.  This, of course, is where mutation testing comes in.

# What is Mutation Testing?

Mutation testing is the process of systematically changing program code to identify when a test or assertion provides insufficient coverage.  I believe it to be best understood by example.  Let's examine some mutants of the function `foo` defined above:

{% highlight cpp %}
// The original function
int foo(int x) {
  return x + 1;
}
// A mutation that can replace the original function
int foo_mutant1(int x) {
  return x - 1;
}
// Another "arithmetic" mutant
int foo_mutant2(int x) {
  return x * 1;
}
// A weirder mutant where we 0 constants
int foo_mutant3(int x) {
  return x + 0;
}
{% endhighlight %}

We refer to each of these variations of this function as "mutants".  Mutants represent changes to the program that _shouldn't work_.  In fact, it is the goal of the program to make every mutant be "killed", meaning that the mutant fails at least one test or assertion.  Mutants that "survive" (are not killed) represent failure of the tests and assertions to properly test for boundaries of the program.

Practically, we can now see how this can be applied to our thought experiment via `foo_mutant1` (or really any of these mutants).  We observe that, when our testing function `bar` calls `foo_mutant1`, the assertion `x > 0` _still holds_ since x = 19 (rather than 21).  In particular, despite the fact that we've mutated (changed) our program, the assertion doesn't fail.  Intuitively, in other words, we've discovered that the test could be tighter around the bound of the program, specifically in that we could have `x == 21` as our assertion.  Note that, if we change the assertion to this value, none of the example mutations survive since every one of them triggers an invalid assertion.

## Why should we care?

While there is something nice about having tight bounds around our tests, why does it matter practically that we do so?  In other words, why bother putting all this time and effort into mutation testing?

In short, mutation testing can improve **test coverage**, meaning that we can improve the amount of code that tests, well, test.  Any testing expert will tell you the value of test coverage; it is (briefly) important to ensure that code is actually tested, and to ensure that breaking changes in code are caught early.  The latter is particularly important from a mutation testing perspective: since mutation testing evaluates potential changes to code, you can catch a lack of testing for breaking changes early.

That being said, mutation testing is not perfect.  There are several flaws to this approach (that we will examine in more detail later) that can make it difficult to implement in a particular domain and impractical for various applications.  Partly as a result of these challenges, mutation testing has yet to become mainstream, instead seeing niche use in various domains of testing and computer science as a whole.

## Why not just regression testing?  Fuzz testing?

For those interested, it is worth examining why we might use mutation testing over the many other methods available for improving testing frameworks and specifically test coverage.  Perhaps most notable among those methods are regression and fuzz testing (or "fuzzing"), both of which see substantial use and are often taught in core university curriculums.  We will first examine a brief overview of these methods before examining when to consider mutation testing as an alternative or even to augment these methods.

Regression testing is most commonly represented by unit tests and broadly refers to having reusable tests that can detect when code is "broken" by changes.  In particular, regression testing captures the most basic unit of correctness of code -- representing a specification in the form of something that must be maintained across changes to the underlying implementation.

Fuzz testing, on the other hand, is the technique of providing random (or more commonly semi-random) input to a program with the goal of discovering bugs that might have not otherwise been tested for.  In this manner, fuzzing can improve test coverage by reasoning about the boundaries of potential inputs to a program and how the program behaves under those conditions.

Unlike these methods, however, mutation testing is focused on the tests themselves, rather than the underlying program.  Specifically, mutation testing examines whether or not test cases can handle broken code that should no longer meet the specification (note that mutation testing generally mutates the program and not the tests themselves).  Notably, this requires that tests already be written, and that the goal of the testing team is to expand testing coverage of code.  In summation, then, mutation testing can _augment_ other testing methods, and should generally be used if the following conditions are true:

* Unit tests or assertions have already been written
* Testing coverage needs to be expanded (or explored)
* Code may potentially change in the future (similar to how regression testing is useful)
* There are sufficiently many tests for mutaiton testing to provide "interesting" results
* There are sufficient resources for extensive testing

In the next sections, we will evaluate some mutation testing theory as well as some practical applications of mutation testing.

part_1 [part_2](http://127.0.0.1:4000/~dgeisler/mutation/testing/2021/11/07/mutation-testing2.html) [part_3](http://127.0.0.1:4000/~dgeisler/mutation/testing/2021/11/08/mutation-testing3.html)