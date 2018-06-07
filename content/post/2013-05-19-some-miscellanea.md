---
id: 510
title: Some miscellanea
date: 2013-05-19T22:39:57+00:00
author: Henry
layout: post
guid: http://the-paper-trail.org/blog/?p=510
permalink: /some-miscellanea/
categories:
  - Distributed systems
  - link
  - Note
---
#### CAP FAQ

I wrote an FAQ on [The CAP Theorem](http://henryr.github.io/cap-faq/)</strong>. The aim is to definitively settle some of the common misconceptions around CAP so as to help prevent its invocation in useless places. If someone says they got around CAP, refer them to the FAQ. It should be a pretty simple introduction to the theorem as well. I think that CAP itself is a pretty uninteresting result, but it does at least shine a light on tradeoffs implicit in distributed systems. I have a couple of residual thoughts about failures rather than partitions that I might write up at some point in the future, but otherwise I hope the FAQ helps move the conversation on.

#### Impala and aggregation trees

I also wrote a quick answer on [Quora](http://qr.ae/pyS1G) about Impala's execution trees, and how deeper trees help do aggregation more effectively. There's a lot more of interest to write about planning, partitioning and scheduling queries.

#### Not so HotOS

Matt Welsh talks about [what he wishes systems researchers would work on](http://matt-welsh.blogspot.com/2013/05/what-i-wish-systems-researchers-would.html). I partially agree: there are few papers in the [HotOS 2013 program](https://www.usenix.org/conference/hotos13/tech-schedule/technical-sessions) that pass the "if this were true, a lot of our assumptions would be wrong" test. I don't think that IO is an unworthy topic however, but the main problem with improving IO interfaces is inertia rather than want for better ideas. I hope that SSDs and / or flash-as-RAM will be forcing functions here. Otherwise his big topics are all worthy research challenges, but I have always seen HotOS as a venue for new ideas, not new topics - despite its name. If it really were intended to be a venue for the fashionable areas of study in systems it would have much less potential value.

#### Building distributed systems

Andy Gross (Basho VP Eng) gave a [closing keynote](https://speakerdeck.com/argv0/lessons-learned-and-questions-raised-from-building-distributed-systems) calling for more reusable primitives in distributed systems. I couldn't agree more (and have been gently complaining about this for years to anyone who would listen), although I think doing this right is not straightforward and requires a lot of careful thought about RPC mechanisms, libraries vs. services and much more. I have some thoughts on this that I want to wrap up in a blog post sometime soon.

#### Highly available transactions

Peter Bailis, a Berkeley PhD candidate, has some solid work on [Highly Available Transactions](http://www.bailis.org/blog/when-is-acid-acid-rarely/), based on exploring the space of consistency models available to databases and finding that there are demonstrably useful consistency guarantees that can be made when availability is kept high. This is the right way to move on from all the CAP arguments - given that we are stuck with certain impossibility results, let's map out the vast terrain that we _can_ conquer.
