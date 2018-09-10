---
id: 290
title: 'CAP confusion: Problems with Partition Tolerance'
date: 2010-04-27T10:44:29+00:00
author: Henry
layout: post
guid: https://the-paper-trail.org/blog/?p=290
permalink: /cap-confusion-problems-with-partition-tolerance/
categories:
  - Distributed systems
  - link
---
Over on the [Cloudera blog](http://www.cloudera.com/blog) I've written an [article](http://www.cloudera.com/blog/2010/04/cap-confusion-problems-with-partition-tolerance/) that should be of interest to readers of this blog. 

I'm no great fan of the ubiquity of the CAP theorem - it's a solid impossibility result which appeals to the theorist in me, but it doesn't capture every fundamental tension in a distributed system. For example: we make our systems distributed across more than one machine usually for reasons of performance and to eliminate a single point of failure. Neither of these motivations are captured verbatim by the CAP theorem. There's more to designing distributed systems!

In this, I agree with Stonebraker; it's the erroneous representation of 'partition tolerance' that I found very strange. I've been a good deal more forceful in private about this than I have in public 🙂
