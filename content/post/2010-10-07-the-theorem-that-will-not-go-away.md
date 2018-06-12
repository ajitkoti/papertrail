---
id: 297
title: The Theorem That Will Not Go Away
date: 2010-10-07T23:28:55+00:00
author: Henry
layout: post
guid: http://the-paper-trail.org/blog/?p=297
permalink: /the-theorem-that-will-not-go-away/
categories:
  - computer science
  - Distributed systems
tags:
  - cap
  - consensus
---
The CAP theorem gets [another airing](http://codahale.com/you-cant-sacrifice-partition-tolerance/).

I think the article makes a point worth making again, and makes it fairly well - that CAP is really about P=> ~(C & A). A couple of things I want to call out though, after a rollicking [discussion](http://news.ycombinator.com/item?id=1768312) on [Hacker News](http://news.ycombinator.com).

> "For a distributed (i.e., multi-node) system to not require partition-tolerance it would have to run on a network which is guaranteed to never drop messages (or even deliver them late) and whose nodes are guaranteed to never die. You and I do not work with these types of systems because they don’t exist."

This is a bit strong, at least theoretically. Actually all you need to not require partition-tolerance is to guarantee that your particular kryptonite failure pattern never occurs. Many protocols are robust to a dropped message here or there. A quorum system requires a fairly dramatic failure (one node completely partitioned) before one side of the partition has to occur. In practice, of course, these failures happen more often than we would like, which is why we worry about fault-tolerant properties of distributed algorithms.

Therefore the paragraph on failure probabilities is less powerful. It's not always a problem if a single failure occurs, and therefore you shouldn't immediately worry about sacrificing availability or consistency as soon as one node starts running slowly. CAP only establishes the _existence_ of a failure pattern that torpedoes any distributed implementation of an atomic object, not its high probability.
