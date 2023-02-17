---
layout: post
title: No one pays me for the Unit Tests
date: '2023-02-17 10:39:38 +0200'
categories: unittests interviews job
---

## From time to time I always hear:
* _"No one pays me for the Unit Tests, so why I should do them, hire a Test Engineer for that!"_
* _"Unit tests are a waste of time."_
* _"We have to write Unit Tests for Unit Tests."_ :D

As a software developer, you've likely heard the argument that unit tests are a waste of time because no one pays you for them. While it's true that unit tests don't directly generate revenue, they play an essential role in software development, and investing time in them can pay off in the long run. In this blog post, we'll explore the importance of unit tests, the benefits they offer developers, and strategies for making time for them.

![Unit Tests Meme](/assets/images/no-one-pays-me-for-the-unit-tests/unit-tests-meme.jpg)

## The Benefits of Unit Tests for Developers
While unit tests don't generate revenue directly, they offer many benefits to developers. By investing time in unit tests, developers can:

* Catch bugs early and reduce the time and effort required to fix them.
* Ensure code correctness and avoid potential issues down the line.
* Enable them to make changes to code with confidence, even in complex or unfamiliar codebases.
* Improve code quality and maintainability by identifying and addressing issues early.
* Write better, more reliable code and improve their coding skills.

## Addressing the "No one pays me" Argument
While it's true that no one pays developers directly for unit tests, it's a flawed argument to say that they're a waste of time. By investing time in unit tests, developers can save time and effort down the line by catching bugs early and avoiding potential issues. Additionally, the benefits of unit tests for developers, such as enabling them to make changes to code with confidence and improving their coding skills, can pay off in the long run.

## The Motivation
For example, [BitMono](https://github.com/sunnamed434/BitMono) had zero lines of code with Tests, but, also when I was looking for a new job in September of 2022 one interviewer was very interested in this project and he asks me a lot, he asked me _"why you don't have Unit Tests in your project?"_ - I said _"I had plans to add Unit Tests in BitMono, however, at the moment I don't see a reason to add them, because the project is in the alpha version, and everything can be changed (the code) in the future."_ In fact, these words are true, tho, I have fooled myself with this answer, I was afraid of adding the Unit Tests to this project, and I was thinking, why, anyway everything works fine. Over time I was thinking about this interviewer's words, and at the moment I'm using practice to add Unit Tests per every feature, if this is possible add 2 Unit Tests to a single feature, also in my opinion it's good practice to write benchmarks to your things in the project, it will help you to understand what's going in the project, and what can be optimized.

The thing that motivates me to write tests is to prove to myself that it works as I think, before tests I was always creating another project (just a console application) to just reproduce some logic and test if it works, also when something is broken and I know that I used this "another console application" to just check why this not work. Now tests show me what works and what does not, and I can simply write another test to check behavior which in my opinion could not work.

### Integrate Unit Tests in Project
It's always a big part of work, to be honest, it may or may not work, but I will try to explain, this is not advice, but I will try to say something useful.

Let's say your team resists the Unit Tests, the team doesn't want to write Unit Tests, have no habit, is not in the mood, no one wants to do it, etc. First of all, a very good phrase, which helps to do a lot, _"Let's write one single test per every feature, for a whole feature."_, just one small simple test for a whole feature, this could sound stupid, but why do we need this one small simple test for a feature, definitely try to share your habit between the team - everyone writing the feature - everyone trying to write the tests, everyone gets used to the tests writing conventions, etc. So, writing the first test is scarier than writing the test the twentieth time. 

You may get this question from the team: _"Which test select to be written?"_ - select by the heart of the programmer, anything you want.
Someone will write sloppy tests, serious tests - it's ok, it still increases the number of tests in your project.

As well it's very important to have CI at this moment.