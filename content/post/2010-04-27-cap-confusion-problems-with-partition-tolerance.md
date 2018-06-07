---
id: 290
title: 'CAP confusion: Problems with Partition Tolerance'
date: 2010-04-27T10:44:29+00:00
author: Henry
layout: post
guid: http://the-paper-trail.org/blog/?p=290
permalink: /cap-confusion-problems-with-partition-tolerance/
categories:
  - Distributed systems
  - link
---
Over on the [Cloudera blog](http://www.cloudera.com/blog) I&#8217;ve written an [article](http://www.cloudera.com/blog/2010/04/cap-confusion-problems-with-partition-tolerance/) that should be of interest to readers of this blog. 

I&#8217;m no great fan of the ubiquity of the CAP theorem &#8211; it&#8217;s a solid impossibility result which appeals to the theorist in me, but it doesn&#8217;t capture every fundamental tension in a distributed system. For example: we make our systems distributed across more than one machine usually for reasons of performance and to eliminate a single point of failure. Neither of these motivations are captured verbatim by the CAP theorem. There&#8217;s more to designing distributed systems!

In this, I agree with Stonebraker; it&#8217;s the erroneous representation of &#8216;partition tolerance&#8217; that I found very strange. I&#8217;ve been a good deal more forceful in private about this than I have in public 🙂