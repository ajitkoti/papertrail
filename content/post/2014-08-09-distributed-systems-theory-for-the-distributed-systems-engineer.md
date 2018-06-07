---
id: 617
title: Distributed systems theory for the distributed systems engineer
date: 2014-08-09T20:45:38+00:00
author: Henry
layout: post
guid: http://the-paper-trail.org/blog/?p=617
permalink: /distributed-systems-theory-for-the-distributed-systems-engineer/
aliases:
 - /blog/distributed-systems-theory-for-the-distributed-systems-engineer/
categories:
  - Distributed systems
---
Gwen Shapira, who at the time was an engineer at Cloudera and now is spreading the Kafka gospel, asked a question on Twitter that got me thinking.

{{< tweet 497203248332165121 >}}

My response of old might have been &#8220;well, here&#8217;s the FLP paper, and here&#8217;s the Paxos paper, and here&#8217;s the Byzantine generals paper&#8230;&#8221;, and I&#8217;d have prescribed a laundry list of primary source material which would have taken at least six months to get through if you rushed. But I&#8217;ve come to thinking that recommending a ton of theoretical papers is often precisely the wrong way to go about learning distributed systems theory (unless you are in a PhD program). Papers are usually deep, usually complex, and require both serious study, and usually _significant experience_ to glean their important contributions and to place them in context. What good is requiring that level of expertise of engineers?

And yet, unfortunately, there&#8217;s a paucity of good &#8216;bridge&#8217; material that summarises, distills and contextualises the important results and ideas in distributed systems theory; particularly material that does so without condescending. Considering that gap lead me to another interesting question:

_What distributed systems theory should a distributed systems engineer know?_

A little theory is, in this case, not such a dangerous thing. So I tried to come up with a list of what I consider the basic concepts that are applicable to my every-day job as a distributed systems engineer; what I consider &#8216;table stakes&#8217; for distributed systems engineers competent enough to design a new system. Let me know what you think I missed!

<!--more-->

### First steps

These four readings do a pretty good job of explaining what about building distributed systems is challenging. Collectively they outline a set of abstract but technical difficulties that the distributed systems engineer has to overcome, and set the stage for the more detailed investigation in later sections

[Distributed Systems for Fun and Profit](http://book.mixu.net/distsys/ "Distributed Systems For Fun and Profit") is a short book which tries to cover some of the basic issues in distributed systems including the role of time and different strategies for replication.

[Notes on distributed systems for young bloods](http://www.somethingsimilar.com/2013/01/14/notes-on-distributed-systems-for-young-bloods/) &#8211; not theory, but a good practical counterbalance to keep the rest of your reading grounded.

[A Note on Distributed Systems](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.41.7628) &#8211; a classic paper on why you can&#8217;t just pretend all remote interactions are like local objects.

[The fallacies of distributed computing](http://en.wikipedia.org/wiki/Fallacies_of_Distributed_Computing) &#8211; 8 fallacies of distributed computing that set the stage for the kinds of things system designers forget.

### Failure and Time

Many difficulties that the distributed systems engineer faces can be blamed on two underlying causes:

  1. processes may fail
  2. there is no good way to tell that they have done so

There is a very deep relationship between what, if anything, processes share about their knowledge of _time_, what failure scenarios are possible to detect, and what algorithms and primitives may be correctly implemented. Most of the time, we assume that two different nodes have absolutely no shared knowledge of what time it is, or how quickly time passes.

You should know:

* The (partial) hierarchy of failure modes: [crash stop -> omission](http://www.cse.psu.edu/~gcao/teach/513-00/c7.pdf) -> [Byzantine](http://en.wikipedia.org/wiki/Byzantine_fault_tolerance). You should understand that what is possible at the top of the hierarchy must be possible at lower levels, and what is impossible at lower levels must be impossible at higher levels.

* How you decide whether an event happened before another event in the absence of any shared clock. This means [Lamport clocks](http://www.stanford.edu/class/cs240/readings/lamport.pdf) and their generalisation to [Vector clocks](http://en.wikipedia.org/wiki/Vector_clock), but also see the [Dynamo paper](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf).

* How big an impact the possibility of even a single failure can actually have on our ability to implement correct distributed systems (see my notes on the FLP result below).

* Different models of time: synchronous, partially synchronous and asynchronous (links coming, when I find a good reference).

### The basic tension of fault tolerance

A system that tolerates some faults without degrading must be able to act as though those faults had not occurred. This means usually that parts of the system must do work redundantly, but doing more work than is absolutely necessary typically carries a cost both in performance and resource consumption. This is the basic tension of adding fault tolerance to a system.

You should know:

* The quorum technique for ensuring single-copy serialisability. See [Skeen&#8217;s original paper](https://ecommons.library.cornell.edu/bitstream/1813/6323/1/82-483.pdf), but perhaps better is [Wikipedia&#8217;s entry](http://en.wikipedia.org/wiki/Quorum_(distributed_computing)).

* About [2-phase-commit](http://the-paper-trail.org/blog/consensus-protocols-two-phase-commit/), [3-phase-commit](http://the-paper-trail.org/blog/consensus-protocols-three-phase-commit/) and [Paxos](http://the-paper-trail.org/blog/consensus-protocols-paxos/), and why they have different fault-tolerance properties.

* How eventual consistency, and other techniques, seek to avoid this tension at the cost of weaker guarantees about system behaviour. The [Dynamo paper](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) is a great place to start, but also Pat Helland&#8217;s classic [Life Beyond Transactions](http://www.ics.uci.edu/~cs223/papers/cidr07p15.pdf) is a must-read.

### Basic primitives

There are few agreed-upon basic building blocks in distributed systems, but more are beginning to emerge. You should know what the following problems are, and where to find a solution for them:

* Leader election (e.g. the [Bully algorithm](http://en.wikipedia.org/wiki/Bully_algorithm))

* Consistent snapshotting (e.g. [this classic paper](http://research.microsoft.com/en-us/um/people/lamport/pubs/chandy.pdf) from Chandy and Lamport)

* Consensus (see the blog posts on 2PC and Paxos above)

* Distributed state machine replication ([Wikipedia](http://en.wikipedia.org/wiki/State_machine_replication) is ok, [Lampson&#8217;s paper](http://research.microsoft.com/en-us/um/people/blampson/58-Consensus/Acrobat.pdf) is canonical but dry).

### Fundamental Results

Some facts just need to be internalised. There are more than this, naturally, but here&#8217;s a flavour:

  * You can&#8217;t implement consistent storage and respond to all requests if you might drop messages between processes. This is the [CAP theorem](http://lpd.epfl.ch/sgilbert/pubs/BrewersConjecture-SigAct.pdf).
  * Consensus is impossible to implement in such a way that it both a) is always correct and b) always terminates if even one machine might fail in an asynchronous system with crash-* stop failures (the FLP result). The first slides &#8211; before the proof gets going &#8211; of my [Papers We Love SF talk](http://www.slideshare.net/HenryRobinson/pwl-nonotes) do a reasonable job of explaining the result, I hope. _Suggestion: there&#8217;s no real need to understand the proof_.
  * Consensus is impossible to solve in fewer than 2 rounds of messages in general

### Real systems

The most important exercise to repeat is to read descriptions of new, real systems, and to critique their design decisions. Do this over and over again. Some suggestions:

#### Google:

[GFS](http://static.googleusercontent.com/media/research.google.com/en/us/archive/gfs-sosp2003.pdf), [Spanner](http://static.googleusercontent.com/media/research.google.com/en/us/archive/spanner-osdi2012.pdf), [F1](http://static.googleusercontent.com/media/research.google.com/en/us/pubs/archive/41344.pdf), [Chubby](http://static.googleusercontent.com/media/research.google.com/en/us/archive/chubby-osdi06.pdf), [BigTable](http://static.googleusercontent.com/media/research.google.com/en/us/archive/bigtable-osdi06.pdf), [MillWheel](http://static.googleusercontent.com/media/research.google.com/en/us/pubs/archive/41378.pdf), [Omega](http://eurosys2013.tudos.org/wp-content/uploads/2013/paper/Schwarzkopf.pdf), [Dapper](http://static.googleusercontent.com/media/research.google.com/en/us/pubs/archive/36356.pdf). [Paxos Made Live](http://www.cs.utexas.edu/users/lorenzo/corsi/cs380d/papers/paper2-1.pdf), The Tail At Scale.

#### Not Google:

[Dryad](http://research.microsoft.com/en-us/projects/dryad/eurosys07.pdf), [Cassandra](https://www.cs.cornell.edu/projects/ladis2009/papers/lakshman-ladis2009.pdf), [Ceph](http://ceph.com/papers/weil-ceph-osdi06.pdf), [RAMCloud](https://ramcloud.stanford.edu/wiki/display/ramcloud/RAMCloud+Papers), [HyperDex](http://hyperdex.org/papers/), [PNUTS](http://www.mpi-sws.org/~druschel/courses/ds/papers/cooper-pnuts.pdf)

### Postscript

If you tame all the concepts and techniques on this list, I&#8217;d [like to talk to you](mailto:henry@cloudera.com) about engineering positions working with the menagerie of distributed systems we curate at Cloudera.
