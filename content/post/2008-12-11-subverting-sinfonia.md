---
id: 106
title: Subverting Sinfonia
date: 2008-12-11T12:59:55+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=106
permalink: /subverting-sinfonia/
categories:
  - Data structures
  - Distributed systems
  - Note
tags:
  - py-concerto
  - sinfonia
---
Recently I presented a paper at a reading group from SOSP &#8217;07 about a transactional distributed shared memory system called [_Sinfonia_](http://www.hpl.hp.com/personal/Mehul_Shah/papers/sosp_2007_aguilera.pdf). The idea is that you given the abstraction of several nodes each with its own contiguous memory address space. You can perform transactions across more than one node at once by locking the set of locations you want to write, then committing or aborting. The paper is quite cute: the main contribution is to restrict the set of transactions to those that can be set up in one message, and therefore piggybacked onto the beginning of a two phase commit, saving a bunch of message latencies.

The idea was sufficiently simple, and I was sufficiently idle that day, that I decided to implement a Sinfonia-like system in Python in a couple of hours. The main point was to test my hypothesis that, at their core, systems presented in research papers aren&#8217;t often complex to understand or implement (this does not speak to their cleverness or value as research contributions). I started at midday, and had my first minitransactions working at around 2:30pm across multiple memory nodes.

Of course, the resultant code was rubbish, so since then (a week or so ago) I&#8217;ve done some tidying up. What results is certainly not top-quality Python by any stretch, but it&#8217;s a bit more workable. There are a lot of features from Sinfonia missing. The most egregious is the lack of a watchdog to shepherd incomplete 2PCs to their conclusion if the client dies &#8211; the expectation is that the memory nodes are sufficiently robust to never fail-stop. Of course, there is no robustness for memory nodes either &#8211; no persistent logging, no self-monitoring and re-start and so on. And there&#8217;s precious little error checking. And bugs are doubtless rife. But it&#8217;s a start, right?

The code is downloadable from [Google Code](http://code.google.com/p/py-concerto/) &#8211; the project is called Py-Concerto. I hope to convert this prototype into much more robust Java soon &#8211; at that point I&#8217;ll stop work on the Python version, but not for a week or two yet.

What kind of stuff can you build on Concerto? There&#8217;s a paper from the same guys who did the original Sinfonia work [here](http://www.hpl.hp.com/techreports/2007/HPL-2007-193.html) on a scalable, distributed B-tree. In general, distributed data structures are a good fit. You could build a reasonably neat overlay network on top of Concerto, using memory locations for pointers and metadata, or a simple distributed linked list.

The code comes with a shell that you can use to talk directly to a Concerto server &#8211; instructions are in the Python scripts themselves.

In the next few days I&#8217;ll add logging and the watchdog process, so watch the svn repo and [let me know](mailto:henry.robinson@gmail.com) if you do anything interesting with this.