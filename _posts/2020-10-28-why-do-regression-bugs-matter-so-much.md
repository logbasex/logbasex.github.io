---
title: 'Why do regression bugs matter so much?'
date: 2020-10-28
permalink: /posts/2020/10/why-do-regression-bugs-matter-so-much/
tags:
  - bugs
  - regression bugs
  - security bugs
  - new feature bugs
---
[//]: # (https://alvinalexander.com/technology/markdown-comments-syntax-not-in-generated-output/)
[//]: # (This post will show up by default. To disable scheduling of future posts, edit `config.yml` and set `future: false`.)
## Why do regression bugs matter so much?

Phone service providers losing data or voice connections. Banks not being able to process transactions. Chaps not being able to transfer funds. SquareSpace not being able to display anyone’s website. What do all these issues have in common?

To start with, they are all issues that sprung to mind when thinking about a blog post about customer issues, which means these are all issues that customers took to social media or mainstream news to complain about.

A customer complaining loudly is bad publicity, so let’s think about why customers go to lengths to give bad publicity when something impacts them. Simply put, it is because it caused them some level of pain by disrupting their daily lives.

 **So what are the important bug types?**

I like to classify bugs into one of three categories: regression, security and new feature. For clarity, a bug is classified as a regression if it is a new bug that changes the behavior of the system negatively. A new feature bug is a new bug that exists entirely within a new feature. A security bug is one that allows an attacker to compromise the system.

Looking at these definitions, I rate the degree of importance to prevent as:
- Regression bugs
- Security bugs
- New feature bugs

**Why are regression bugs so important to prevent?**

By now, I am sure that you are thinking of a company or product you use that has impacted you through a bug introduced during an upgrade. Essentially this eroded trust between you and the vendor. For business-to-business relationships, this could be enough to cause financial penalties in the contract, or even mean that the client looks around for a new vendor.

These are the bugs that convert to severity zero/one customer issues very quickly. When a customer can no longer do their job, they will rightly expect your development and support teams to be sweating while trying to fix their issues.

**Why are security bugs in second place?**

These bugs are sexy and newsworthy. We are all quick to judge when a company loses our personal data, and this subject is very often jumped on by the mainstream news. Large companies like TalkTalk, Marriott, and Yahoo have all been in the news over the last few years for security vulnerabilities. Nobody wants to have their information stolen or leaked, and this type of bug can also erode trust, which is why it’s in second place.

**New features, last place?!?**

When a company launches a new feature with a bug in it, what happens? The customer tries the new feature, decides it doesn’t work and goes back to whatever they were doing before. This is why these bugs are less serious; they don’t stop the customer from doing what they were already accustomed to doing.

I challenge you to send me a link to a mainstream news article talking about a software bug in an entirely newly released feature. (I haven’t been able to find one).

**If regressions are so important, why don’t we test them more?**

It’s boring! Trust me, by the third release where you are testing the same old feature manually, you are bored of it. It rarely holds much excitement for the engineer doing the testing. Management is only interested if you find a serious bug, and even then it is an inconvenience to the timetable.

Therefore, most testers will spend the majority of their time working on testing new features. This is the fun and exciting area of working in a QA team: Being able to try and shape the product before anyone else.

**What’s the answer?**

Automation, automation and a little more automation. Finding regressions is exactly the task that automated testing can do. If you want to stop these bugs causing pain across your organization then the answer is simple: ensure that your regression coverage is adequate.

This solution also sounds like it has been around for the entirety of my career. It is time to stop begging management for the time to do more automated testing and start using [tools that can use AI for code](https://www.diffblue.com/products) to make your automated testing faster to implement.

Reference: https://medium.com/@DiffblueHQ/why-do-regression-bugs-matter-so-much-f5fd931c1670

