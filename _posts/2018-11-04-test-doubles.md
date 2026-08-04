---
layout: post
title: "Test Doubles"
date: 2018-11-04 00:00:00 +0000
tags: ["test doubles", "testing", "testing strategy", "tests"]
---

Sometimes when writing tests or defining a testing strategy, we need to replace part of a system at run time with a modified version, in order to set up, simplify, isolate or verify a particular behaviour.

In his book [xUnit Test Patterns: Refactoring Test Code](https://www.goodreads.com/book/show/337302.xUnit_Test_Patterns), Gerard Meszaros came up with the term test doubles to describe the various types of objects or procedures used to facilitate and support testing:

- **Mocks** are used to verify behaviour by throwing exceptions or failing when they receive calls that differ from their expectations.
- **Spies** are stubs that record some information based on how they were called. They are generally used when we need to verify if/how many times a function call has been made.
- **Stubs** provide predictible responses to the calls made during the test, usually programmed to only respond to calls that happen in the context of the test.
- **Fake objects** have working implementations, but are not suitable for production, given they might have some implementation shortcuts. In-memory database is a common example.
- **Dummy objects** are passed around but never used. They are commonly used to fill parameter lists, fill in required properties in other objects, etc.

More on this topic:

- [Test double](https://en.wikipedia.org/wiki/Test_double)
- [Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
