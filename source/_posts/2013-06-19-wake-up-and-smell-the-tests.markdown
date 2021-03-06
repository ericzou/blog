---
layout: post
title: "Wake up and smell the (slow) tests"
date: 2013-06-19 09:01
comments: true
categories: [Testing, Rails]
---

I've been on many projects where writing/running tests were painfully slow.  To test one line of change often requires me to construct a bunch of unrelated models, fix a few failures in other tests and then patiently wait an hour or two for the tests suite to pass locally and on CI.

Even worse, the developers are often proud of themselves for writing the tests first to have an excellent test coverage.  These painfully slow processes for writing/running tests are often accepted as the cost of doing TDD.

It is not. These painfully slow tests are a clear hint that the code/test can be structured better.  Rather than blindly writing more tests, I usually ask myself the following questions.

### Do my tests have too many inter-process communications?
Inter-process communications are an order of magnitutue slower than in-memory communications.  Mock out the interfaces if that's the case.  Better yet, try to design a system where these external connections are explicitly inversely injected.  Then during the test all is needed is just to swap in a fake connection.

### Am I testing at the right level?
Integration tests are a necessary evil and need to be treated as such.  Writing an integration test for an 'if' statement change in the model can often be inappropriate.  Unfortunately, many developers thinks
the more the better when writing integration tests, which often leads to a bloated test suite.

### Is my test setup complicated?
If I have to create lots of objects to be able to test my object then that's usually a sign the objects are too coupled together.  I've seen this happen to a lot in Rails projects since the framework itself does very little to prevent this.  If I spend more time setting up the tests than actually writing it, I would investigate it a bit to see if I can decouple the code dependencies.

### Can I modulize part of the project?
Modulized code base means a decoupled structure, which leads to a simplier test setup, smaller test suites, and faster test runs.

### Are my tests too 'meta'
Writing tests that generates more tests seems clever but there actually lots of problems:

* Very easy to go overboard with it:
In one of my projects, we went 'meta' for generating access control tests - a test for every role (guest, user, admin, etc), and every path combination to make sure current users can/cannot access certain pages. That alone, generated thousands of tests and took a very long time to run.

* Lots of valueless tests:
With tests like these, we often try to test every possible scenario. but when we really only care about the common cases and edge cases.

* Too abstract:
The intent of each test should be very clear and easy to read but that's usually not the case when tests are 'meta'.

* Hard to debug:
It's often hard to run a single generated test without running the entire thing and the line numbers are usually wrong.

### Does my test code look boring or repetitive?
Tests should be interesting to write.  If they look repetitive, that usually signals repeated usage of code or code that have a common pattern with slight variations.  If this occurs, I would review and try to see if I can extract the common pattern out into a behavior and create a separate test just for that behavior.

