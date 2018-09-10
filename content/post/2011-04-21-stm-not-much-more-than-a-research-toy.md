---
id: 311
title: 'STM: Not (much more than) a research toy?'
date: 2011-04-21T13:02:38+00:00
author: Henry
layout: post
guid: https://the-paper-trail.org/blog/?p=311
permalink: /stm-not-much-more-than-a-research-toy/
categories:
  - Operating systems
---
It's a sign of how down-trodden the Software Transactional Memory (STM) effort must have become that the [article](http://cacm.acm.org/magazines/2011/4/106585-why-stm-can-be-more-than-a-research-toy/fulltext) (sorry, ACM subscription required) published in a recent CACM might have been just as correctly called "STM: Not as bad as the worst possible case". The authors present a series of experiments that demonstrate that highly concurrent STM code beats _sequential, single threaded code_. You'd hope that this had long ago become a given, but what this demonstrates is only hey, STM allows _some_ parallelism. And this weak lower bound got a whole article.

Another conclusion from the article is that STM performs best when there is little contention for transactions between threads. Again, that should really be a given - all reasonable concurrency primitives have high throughput when there is little contention but high parallelism. (A lot of work has gone into making this a very fast case (since it is the most common) for locking, see e.g. [biased locking schemes](http://blogs.sun.com/dave/entry/biased_locking_in_hotspot) in the Hotspot JVM).

Bryan Cantrill (previously of Fishworks, now of Joyent) [rips on transactional memory](http://blogs.sun.com/bmc/entry/concurrency_s_shysters) more eloquently than I ever could. STM is a declarative solution to thread safety, which I like, but no more declarative really than synchronised blocks - and Cantrill points out the elephant in the room that the CACM article seemed to ignore: doing IO inside transactions is hugely problematic (because how precisely do you roll back a network packet?).

A recent paper at SOSP 2009 called [Operating System Transactions](http://www.sigops.org/sosp/sosp09/papers/porter-sosp09.pdf) attacked this problem, although not from the viewpoint of STM, but to provide atomicity and isolation for situations where bugs arise from the separation between reads, and writes that depend on that read (Time Of Check To Time Of Use - TOCTTOU). Perhaps there's an overlap between this paper and STM approaches, but it's not clear whether the workloads inside an operating system's system call layer are general enough to map onto typical user-space STM work.
