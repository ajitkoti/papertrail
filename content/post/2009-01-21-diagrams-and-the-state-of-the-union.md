---
id: 150
title: Diagrams, and the state of the union
date: 2009-01-21T17:29:16+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=150
permalink: /diagrams-and-the-state-of-the-union/
categories:
  - Note
tags:
  - meta
  - Note
---
Due to popular request, I&#8217;ve started retrospectively adding some diagrams to articles that really need them. First to get the treatment has been [two-phase commit](http://hnr.dnsalias.net/wordpress/?p=90) &#8211; by far the most popular article on this blog. The [Dynamo](http://hnr.dnsalias.net/wordpress/?p=51) article will be the next up, then 3PC and maybe the GFS and BigTable entries.

I&#8217;m using an old version of [OmniGraffle](http://www.omnigroup.com/applications/OmniGraffle/), which came installed on my Powerbook G4. I recently replaced the power supply board in the G4 (which involved some hair-raising open Mac surgery) and am delighting in having all these great applications at my fingertips again. OmniGraffle makes diagrams for things like this so very easy. The effort expended in producing a diagram is far less than that for writing 1000 words, so if the old adage is true this is a very efficient way of producing content.

Although Real Work is consuming a lot of my time at the moment, I&#8217;ve been pretty good at finding time here and there to write for this blog. I&#8217;ve got two outward-facing goals here. The first is to make available some clear explanations for basic distributed systems theory and practice. I think there&#8217;s a niche for good work here &#8211; I don&#8217;t think any textbooks I have read adequately treat practice in a sufficiently theoretical way, and the theory textbooks can be too abstract to be accessible. Therefore I&#8217;ve been writing articles like [the tour of FLP impossibility](http://hnr.dnsalias.net/wordpress/?p=49), the aforementioned introduction to [two-phase](http://hnr.dnsalias.net/wordpress/?p=90) and [three-phase](http://hnr.dnsalias.net/wordpress/?p=103) commit and the discussion of [consensus in the context of lossy links](http://hnr.dnsalias.net/wordpress/?p=110). Continuing in this vein, I have plans to talk about [Paxos](http://en.wikipedia.org/wiki/Paxos_algorithm), failure detectors, distributed spanner construction and some more simple, fundamental distributed algorithms such as leadership election.

My second &#8216;public-facing&#8217; goal is to survey some of the more interesting (and occasionally less interesting) systems research, with a particular emphasis on real systems that exist and work. Hence the [GFS](http://hnr.dnsalias.net/wordpress/?p=58), [BigTable](http://hnr.dnsalias.net/wordpress/?p=86) and [PNUTS](http://hnr.dnsalias.net/wordpress/?p=80) articles, and the recent series on [OSDI](http://www.usenix.org/events/osdi08/) papers. Part of my day job is being familiar with OSDI, NSDI, SOSP, HotOS, Mobi* etc. conferences and workshops, and by writing the articles I get the chance to consolidate my understanding, which is highly useful.

I&#8217;d be very interested to hear, by [mail](mailto:henry.robinson@gmail.com) or by comment, if there are any particular topics that you&#8217;d like me to cover. I suspect the imminent article on Paxos will be popular (executive summary: it&#8217;s not that hard, especially if you already understand 3PC), but otherwise it&#8217;s hard to gauge what people are looking forward to reading on this blog. I even would enjoy picking up writing about algorithms that I abortively [started to do](http://hnr.dnsalias.net/wordpress/?p=46). So help me out!