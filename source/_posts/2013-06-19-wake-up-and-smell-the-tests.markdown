---
layout: post
title: "Wake up and smell the (slow) tests"
date: 2013-06-19 09:01
comments: true
categories:
---

I've been on many projects where writing/runing tests were painfully slow. To test one line of change often requires me to construct a bunch unrelated models, fix a few failures in other tests and patiently wait an hour or two for the tests suite to pass locally and on ci.

Worse, Developers often proud themselves for writing tests first, and have an excellent test coverage. These painfully slow processes for writing/running tests are often accepted as the cost of doing TDD.

It is not. If the test is painful to write and slow to run. It's a smell that our code/test can be structured better. Rather than blindly writing more tests, I usually ask myself the following questions.


### Do my tests have too many inter-process communications?
Inter-process communications are an order of magnitutue slower than in-memory communications. Mock out the interfaces if that's the case. Better yet, try design a system where these external connections are explicitly inversely injected. Then during the test, I just need to swap in a fake connection.

### Am I testing at the right level?
Intergration tests are a necessary evil and need to be treated as such. Writing an intergration test for an if statement change in the model can often be inapproproate. Unfortunately, integration tests, once in place provides a false sense of security and are usually difficult to remove. So it's important to be very causious about them from the very beginning.

### Is my test setup complicated?
If I have to create lots of objects to able to test my object. That's usually a sign the objects are too coupled together. I've seen this happen a lot in Rails projects, since the framework itself does very little to prevent this.  If I spend more time setting up the tests than actually writing it, I would investigate it a bit to see if I can decouple the code dependencies.

### Can I modulize part of the project?
Modulized code base means a decoupled structure, which leads to a simplier test setup, smaller test suites, and faster test runs.

### Is my tests too 'meta'
Writing tests that generates more tests seems clever. But it sufer from a number of problems:
* Easily go overboard with it: In one of my projects, we went 'meta' for generating access control tests - a test for every role(guess, user, admin, etc), and every path to combination to make sure current user can/cannot access certain pages. That alone generated thousands of tests and took a long time to run.

* Lot of valueless tests: When we have tests like these, we are often testing every possible scenario. When we really care only the common cases and edge cases.
* Too abstract: The intent of the each tests should be very clear and easy to read, but that's usually not the case when tests are 'meta'
* Hard to debug: It's often hard to run a single generated test without running the entire thing, and the line numbers are usually wrong.

### Does my test code look boring or repetitive?
Tests should be interesting to write. If they look repetitive, that usually signals the userage of the code will have a common pattern with slight variarations. I would reconsider see if I can abstract the common pattern out into a behavior and a separate test for for that behavior.

## Final Thought
Many of us have grown custom to these slow and painful tests. Many of us have followed the TDD without really understand its spirit.  Maybe its time to wake up and smell the tests, they are trying to tell you something.


