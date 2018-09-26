---
title: "Paper Notes: MICA: A Holistic Approach to Fast In-Memory Key-Value Storage"
date: 2018-09-26T12:52:31-07:00
author: Henry
permalink: /paper-notes-mica/
draft: false
layout: post
categories:
    - Paper notes
    - Key-value stores
---

### MICA: A Holistic Approach to Fast In-Memory Key-Value Storage

_Lim et. al., NSDI 2014_
\[[paper](https://pdos.csail.mit.edu/papers/masstree:eurosys12.pdf), [code](https://github.com/kohler/masstree-beta)\]

In this installment we're going to look at a paper from NSDI 2014

<!--more-->

##### The Big Idea

MICA is another key-value store, one that starts from a different set of assumptions to, for
example, Masstree, and as a result ends up at a rather different point in the design space. The
target environment is pretty similar: MICA is designed to take advantage of multi-core CPUs, and all
keys and values are stored in memory. Storage is limited to a single node - there's no replication
or partitioning (although those could probably be added without changing much of MICA as presented
here). The data stored are string `(key, value)` pairs. But beyond those initial similarities, there
are some important differences:

**No range queries**: MICA supports `get(key)`, `put(key, value)` and `delete(key)` operations, but
does not support `getRange(low_key, high_key)`. This is a critical difference: if you don't have to
service range queries you don't have to index any information about the _relative_ values of keys,
and therefore you don't have to perform many comparisons on the read or write paths. In Masstree we
saw that a lot of effort went into optimizing comparisons by keeping them constant-cost. That
shouldn't be a focus for a system like MICA.

**Emphasis on cache semantics**: MICA supports four operating modes (more on that later). The
largest focus in the paper is on _cache_ semantics, rather than _store_ semantics, although
implementations for both are provided. Cache semantics are easier to support than store semantics,
because MICA can choose what data to retain in order to preserve performance and manage memory
usage.

**Keys and values are short**: Masstree focused on long keys and values with potentially a high
degree of prefix overlap. MICA instead assumes that keys and values are relatively short - short
enough to fit into a UDP packet, so about 64k in total. Shorter keys and values present their own
difficulties: comparison costs are lower so perhaps not so important to focus on, but they also
require small memory allocations which can lead to more fragmentation and higher overhead (e.g. the
fixed cost of tracking an allocation becomes more significant with higher allocation volume).

As far as performance goals go, MICA targets low end-to-end latency, consistent performance across
workloads and high single-node throughput. How does it go about achieving them? Let's find out.

##### Data structures, logs and hash tables



##### Parallelism: partitioning vs coordination

One of the major design decisions in MICA is to achieve parallelism by partitionining work across
multiple cores. Each core maintains its own private data structures and doesn't need to read or
write any other cores' data. If done correctly, this can be an embarassingly parallel implementation
with little or no coordination required between cores, but the potential downside is that, since
each operation can only be processed by one core, any imbalance in workload will lead to imbalanced
core utilization and poor efficiency. The opposite philosophy is to allow any core to service any
request and to embrace the coordination that this requires; Masstree took this approach.

Workload skew is pretty much an accepted fact of life, so how does MICA justify or mitigate the
per-core imbalance that results? The answer is in section 4.1.1. Firstly keys are _hashed_ to map
them to a core; this can spread the load from a popular range of keys, but does nothing when one key
gets particularly hot.

MICA can take advantage of a natural optimization that happens when load on a particular partition
is high: the network layer can **batch** delivery of many operations at once (up to 32 at a
time). Apparently the per-packet overhead of managing the delivery queue is high, so amortizing that
across 32 packets at once is a significant win.

It's hard to know how reasonable these claims are. There's an interplay between CPU and network
efficiency which isn't really called out. In order for batching to be effective, requests have to
arrive more frequently than the CPU can read them from the network queue. So batching doesn't help
until the core is tapped out.

The point being that you could have a core processing 100 requests/s or more because of your amazing
optimized data structures, and the observed latency is still lower than if those requests were
spread across multiple cores (at perhaps a slightly higher cost / operation).

If the CPU is slower than the arrival rate, then batching causes the per-operation cost to go down
(the network queue interaction cost is divided by 32). So the throughput of that core goes up, but
the increase has to be larger than the ratio of workload it is receiving in order for it to break
even (if the only CPU processing that was done was to read a packet from the network queue, the
throughput would increase by 32x, therefore this is a win if a non-batched core is processing 32x
fewer operations or more).

The evaluation of MICA shows that Zipfian skew causes throughput per-core to drop as the number of
cores increases, but not precipitously up to 15 cores. An interesting data point is that per-core
throughput is higher for the skewed load at one core, which is presumably the recipient of most
load. Assuming that the cores were being 100% used, this points to the benefits of data locality
when serving skewed workloads. The aggregate throughput across all cores, however, is lower for the
skewed workload.

##### Partitioning without coordination

It's all well and good to have data structures that don't need coordination, but you need a way

##### How is this better than a hash table?
