---
layout: post
title: Working effectively with legacy code - Michael C. Feathers
categories: [java, development]
tags: [java, testing, legacy]
---

This is an article about the book `Working Effectively with Legacy Code` by Michael C. Feathers and my thoughts about it.

# Legacy Code

This is a well known definition of Michael C. Feathers in his book `Working Effectively with Legacy Code`.

> Code without tests is bad code. It doesn't matter how well written it is; it doesn't matter how pretty or object-oriented or well-encapsulated it is. With tests, we can change the behavior of our code quickly and verifiably. Without them, we really don't know if our code is getting better or worse.

For me `Legacy Code` is not untested code but unmaintainable code.
Maintainable code is more than just  tested code.
I forgot where I read the statement like

> Once in the past a experienced developer was someone who has written codes no one could understand. Today an experienced developer is writing code even the beginners can understand.

For sure, you should have tests to prove your code will also work in years an intended. Maintainable code will be readable and is tested in most cases. Let me describe what I learned from this book so far.

# Goal

The goal of this book is to show principles and ways to make `Legacy Code` testable.
Michael C. feathers describes several principles and techniques you can use to reach the goal.

The next chapters will be no abstract of the book but what came in my mind while reading it to use in the future.

# Techniques

## Test Driven Development aka TDD

Well known technique for writing and testing new code.

- write failing test
- make it pass
- refactor
- repeat

## Characterization Test

A way to write tests for `Legacy Code`.

- write failing test for `Legacy Code`
- let the failure tell you what the behavior is
- change the test to make it pass
- repeat

## Sprout Method/Class

From the book chapter 6 `I Don't Have Much Time and I Have to Change It`.

The main idea is to write new code in new methods or classes and invoke them in the right places.

The advantages are that all _good_ principles like readable code and TDD can be applied. The disadvantage is that old code is not affected and therefore no improvement happened.

## Wrap Method/Class

Also from the book chapter 6 `I Don't Have Much Time and I Have to Change It`

In this case it describes the implementation of the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle) (one responsibility per method class).

## Dependency Injection

A main concern are the dependencies which sometimes makes it hard to write tests.

### Constructor Injection

If you don't use a dependency framework like spring, guice or others one way is to use constructor injection.

```java
public MailSender() {
  this(new DefaultMailServer());
}

public MailSender(MailServer mailServer) {
  this.mailServer = mailServer;
}
```

The second constructor can be used by tests to mock the _MailServer_.

### Enhanced constructor injection with builder pattern

The builder pattern could be a good solution to get rid of all variations of constructors if there is the need to inject more than just one dependency.

```java
createMailSender()
  .withMailServer(new DefaultMailServer())
  .withEncoding(HTML)
  .build();
```

## Breaking Dependencies

Often it is very hard to write tests because of all these dependencies. Here are some solutions to break them.

- Wrap method parameters of (3rd party) dependencies to have a stable interface without the dependency to the framework used
- Move methods into new classes
 - pass parameters to constructor
 - one method like `run()` or `execute()`
- Access globals through regular classes

# Principles

Some principles mentioned in the book are now known as [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) principles. These can be read all over the Internet.

## Do it now or you will never do it

A common problem of real life is that there is no second time to do it right.

## Try to write tests to see if it is possible

Even the longest journey starts with the first step. You have to make it, otherwise you will never know if it is possible to write tests for `Legacy Code`.

## Start with easy tests

To make your life easier start to write tests for simple examples or some code you know how it it working.

## Command and Query Principle

A _command_ only modifies a state but has no return value. A _query_ does not modify any state but has a return value.

The principle is to avoid side-effects inside methods which can be major pain while trying to write tests. And it is also a good example of SRP.

## Use your IDE for refactoring

All major IDEs offer the possibility to refactor code in a safe way. Don't do it manually. Even if you have years of experience, there will be one refactoring that failed and you have to look for the book for hours.
