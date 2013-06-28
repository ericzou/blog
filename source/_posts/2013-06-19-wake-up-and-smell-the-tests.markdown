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
Intergration tests are a necessary evil but need to be treated with caution.Writing an intergration test for an if statement change in the model can often be inapproproate. This is trading productivity for a false sense of security. Unfortunately, integration tests, once in place, are very difficult to remove since emotionally they provide a false sense of security that many developers relies on. So it's important to be very causious about them from the very beginning

### Is my test setup complicated?
If I have to create lots of objects to just able to that's usually a sign the models are too coupled together. I've seen this happen a lot in Rails projects, since the framework itself does very little to prevent this.  If I spend more time setting up the tests than actually writing it, I would investigate it a bit to see if I can decouple the code dependencies.

### Can I modulize part of the project?
Modulized code base usually has a decoupled code structure, which leads to a simplier test setup, smaller test suites, and faster test runs.

### Is my tests too 'meta'
Writing tests that generates more tests seems clever. But it sufer from a number of problems:
* Easily go overboard with it: In one of my projects, we went 'meta' for generating access control tests: that is a test for every role(guess, user, admin, etc), and every path to combination to make sure current user can/cannot access certain pages. That alone generated thousands of tests.

* Lot of valueless tests: When you have tests like that, you are testing every possible values. When we really care only the common and edge cases.

* Abstract: The intent of the each tests should be very clear and easy to read, but that's not usually the case when tests are 'meta'
* Hard to debug: It's often hard to run a single test without running the whole


* Tests code looks boring and repetitive
* Tests are difficult to write


* Does my test code look boring or repetitive? Ok, this one might not contribute to the slowness of the tests, but its a smell usually related to my implementation.



# Smell 1: Your tests run slow
This usually is the most important indicator to watch out for. Almost always, slow tests tells you something is wrong in your code - maybe your test code is too repetitive, maybe your are constructing too many objects in tests. If leave untreated, the slow tests will kill you productivity and destroy the benefit of TDD.

## Your tests code looks repetitive.
That usually is a sign that some of the funcationality can be extracted out into modules, and you should really test these individual modules rather than the objects that use these modules. Sometimes, we generate tests to handle repetitive testing, something like this:

```ruby
[ChildClass1, ChildClass2].each do |subject|

  describe "#{subject}" do
    it "should do something here" do
      # expects...
    end
  end

end
```

Generated tests usually does more bad than good. Because they are hard to debug(the line numbers are usually wrong), hard to run individually, and mask the real problem - your code needs to be more moduluar and these modules themselves need to be testable.


## You have to construct a bunch unrelated modules in order to test your code
That's a sign your models are too coupled with others. This is a common sin for ActiveRecord. Imaging the following code:

```ruby
  class User < ActiveRecord

    # contrived example: get the tasks assigned to a specific role that the user belongs to
    def tasks
      user.roles.first.assignments.first.task
    end

  end
```

In order to test this, you will need to construct the User, a few Roles, a few Assignments, and a Task. Yuk. To make the matter worse, these dependencies are introduced arbitrarly, made it very hard to change the code.

This is one of the main reason people are calling out for places otherside of ActiveRecord to handle business heavy logics.


```ruby
  class TaskManager
    def
  end
You can usually cure this by explicitly declare dependencies, and inject them into your model(Dependency Injection).
