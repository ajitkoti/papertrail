---
id: 604
title: 'The Elephant was a Trojan Horse: On the Death of Map-Reduce at Google'
date: 2014-06-25T17:49:39+00:00
author: Henry
layout: post
guid: https://the-paper-trail.org/blog/?p=604
permalink: /the-elephant-was-a-trojan-horse-on-the-death-of-map-reduce-at-google/
categories:
  - Uncategorized
---
*Note: this is a personal blog post, and doesn't reflect the views of my employers at Cloudera*

**Map-Reduce is on its way out. But we shouldn't measure its importance in the number of bytes it crunches, but the fundamental shift in data processing architectures it helped popularise.**

This morning, at their I/O Conference, Google revealed that they’re [not using Map-Reduce to process data internally at all any more](http://www.datacenterknowledge.com/archives/2014/06/25/google-dumps-mapreduce-favor-new-hyper-scale-analytics-system/).

We shouldn’t be surprised. The writing has been on the wall for Map-Reduce for some time. The truth is that Map-Reduce as a processing paradigm continues to be severely restrictive, and is no more than a subset of richer processing systems.

<!--more-->

It was known for decades that generalised dataflow engines adequately capture the map-reduce model as a fairly trivial special case. However, there was real doubt over whether such engines could be efficiently implemented on large-scale cluster computers. But ever since [Dryad, in 2007](http://research.microsoft.com/en-us/projects/dryad/) (at least), it was clear to me that Map-Reduce's days were numbered. Indeed, it's a bit of a surprise to me that it lasted this long.

Map-Reduce has served a great purpose, though: many, many companies, research labs and individuals are successfully bringing Map-Reduce to bear on problems to which it is suited: brute-force processing with an optional aggregation. But more important in the longer term, to my mind, is the way that Map-Reduce provided the justification for re-evaluating the ways in which large-scale data processing platforms are built (and purchased!).

If we are in a data revolution right now, the computational advance that made it possible was **not** the ‘discovery’ of Map-Reduce, but instead the realisation that these computing systems can and should be built from relatively cheap, shared-nothing machines (and the real contribution from Google in this area was arguably [GFS](http://static.googleusercontent.com/media/research.google.com/en/us/archive/gfs-sosp2003.pdf), not Map-Reduce).

The advantages of this new architecture are enormous and well understood: storage and compute become incrementally scalable, heterogeneous workloads are better supported and the faults that more commonly arise when you have ‘cheaper’ commodity components are at the same time much easier to absorb (paradoxically, it’s easier to build robust, fault-tolerant systems from unreliable components). Of course lower cost is a huge advantage as well, and is the one that has established vendors stuck between the rock of having to cannibalise their own hardware margins, and the hard place of being outmaneuvered by the new technology.

In the public domain, Hadoop would not have had any success without Map-Reduce to sell it. Until the open-source community developed the maturity to build successful replacements, commodity distributed computing needed an app - not a ‘killer’ app, necessarily, but some new approach that made some of the theoretical promise real. Buying into Map-Reduce meant buying into the platform.

Now we are much closer to delivering much more fully on the software promise. MPP database concepts, far from being completely incompatible with large shared-nothing deployments, are becoming more and more applicable as we develop a [better understanding of the way to integrate distributed and local execution models](http://research.microsoft.com/en-us/um/people/jrzhou/pub/scope-vldbj.pdf). I remember sitting in a reading group at Microsoft Research in Cambridge as we discussed whether joins could ever be efficient in cluster computing. The answer has turned out to be yes, and the techniques were already known at the time. Transactions are similarly thought to be in the ‘can never work’ camp, but [Spanner has shown that there’s progress to be made in that area](http://research.google.com/archive/spanner.html). Perhaps OLTP will never move wholesale to cluster computing, but data in those clusters need not be read-only.

As these more general frameworks improve, they subsume Map-Reduce and make its shortcomings more evident. Map-Reduce has never been an easy paradigm to write new programs for, if only because the mapping between your problem and the rigid two-phase topology is rarely obvious. Languages can only mask that impedance mismatch to a certain extent. Map-Reduce, as implemented, typically has substantial overhead attributable both to its inherent ‘batchness’, and the need to have a barrier between the map and reduce phases. It's a relief to offer end-users a better alternative

So it’s no surprise to hear that Google have retired Map-Reduce. It will also be no surprise to me when, eventually, Hadoop does the same, and the elephant is finally given its dotage.
