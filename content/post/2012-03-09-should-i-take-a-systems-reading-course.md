---
id: 342
title: Should I take a systems reading course?
date: 2012-03-09T18:05:13+00:00
author: Henry
layout: post
guid: http://the-paper-trail.org/blog/?p=342
permalink: /should-i-take-a-systems-reading-course/
categories:
  - Uncategorized
---
A smart student asked me a couple of days ago whether I thought taking a 2xx-level reading course in operating systems was a good idea. The student, understandably, was unsure whether talking about these systems was as valuable as actually building them, and also whether, since his primary interest is in &#8216;distributed&#8217; systems, he stood to benefit from a deep understanding of things like virtual memory. 

<!--more-->

I figured, since I get a bunch of e-mail from students through this site, that it might be worth sharing my answer:

> Take it, if you&#8217;re serious about distributed systems, and here&#8217;s why:
> 
> Systems &#8216;thinking&#8217; is terribly important. One way to learn that well is to read papers and to discuss them. This hones the ability to critically think about computer systems, about the value of certain lines of work and about the meaning (or lack of) of performance results. In the absence of a formalised way of judging systems work (i.e. through a proof) you need to develop your own judgment and taste. This is a very good way to do so.
> 
> Not only that, the topics in the course are practically very relevant. Distributed systems aren&#8217;t any different from other systems topics in the sense that they interact with the network, disk and CPU in much the same way. To give you an example, distributed resource management (i.e. Mesos or YARN) which is an incredibly relevant &#8216;distributed&#8217; systems topic right now needs to understand local scheduling policies, the relative characteristics of disk and memory and the interaction between CPU and the memory hierarchy to begin to be effective.
> 
> Thinking of distributed systems as an abstraction over a collection of nodes works best only when you are thinking about distributed algorithms, but it doesn&#8217;t work if you&#8217;re an engineer. If you&#8217;re mathematically minded, and you&#8217;re not that keen on actually implementing what you come up with instead of proving something about it, then this might not be the course for you (although I&#8217;d \*still\* suggest you consider it strongly).
> 
> But if you like to build things, and you like to \*really\* understand how they work, you&#8217;re going to need to know all of this stuff and more.