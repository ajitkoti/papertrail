---
id: 50
title: 'Dijkstra award 2008 goes to &#039;Sparse Partitions&#039;'
date: 2008-08-17T19:59:28+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=50
permalink: /dijkstra-award-2008-goes-to-sparse-partitions/
categories:
  - Distributed systems
---
Not sure why I didn&#8217;t post this when I first found out &#8211; the Djikstra 2008 prize for an outstanding and influential paper in distributed systems has been awarded to [David Peleg](http://www.wisdom.weizmann.ac.il/~peleg/) and [Baruch Awerbuch](http://www.cs.jhu.edu/~baruch/) for their 1990 FOCS paper _Sparse Partitions_ (on-line copy available from MIT ad-hoc algorithms course [here](http://theory.csail.mit.edu/classes/6.885/spring06/papers/AwerbuchPeleg-focs.pdf)).

The [citation](http://www.podc.org/dijkstra/2008.html) does a much better job than I could of explaining the paper&#8217;s relevance. The general idea is that the authors show that there are efficient ways of constructing clustered representations of graphs that remain within a small factor of the original in terms of route lengths. Further, the authors show that this can be done in a distributed manner. This has lots (and lots) of potential applications &#8211; the typical example is for a compact routing scheme, where nodes can store smaller routing tables between clusters rather than between nodes.

I&#8217;ve got the paper cued up on my list of walkthroughs to write, so expect a better explanation than the one above soon.