---
id: 595
title: 'Paper notes: MemC3, a better Memcached'
date: 2014-06-18T14:36:51+00:00
author: Henry
layout: post
guid: http://the-paper-trail.org/blog/?p=595
permalink: /paper-notes-memc3-a-better-memcached/
categories:
  - Paper notes
---

### MemC3: Compact and Concurrent MemCache with Dumber Caching and Smarter Hashing

  _Fan and Andersen, [NSDI 2013](https://www.usenix.org/conference/nsdi13/)_

#### The big idea

This is a paper about choosing your data structures and algorithms carefully. By paying careful attention to the workload and functional requirements, the authors reimplement [memcached](http://memcached.org/) to achieve a) better concurrency and b) better space efficiency. Specifically, they introduce a variant of cuckoo hashing that is highly amenable to concurrent workloads, and integrate the venerable CLOCK cache eviction algorithm with the hash table for space-efficient approximate LRU.

<!--more-->

#### Optimistic cuckoo hashing

[Cuckoo hashing](http://en.wikipedia.org/wiki/Cuckoo_hashing) has some problems under high-concurrency. Since each displacement requires accessing two buckets, careful locking is required to make sure that updates don't conflict, or worse deadlock since you need to lock the entire path through the hash-table on each insert. Without this locking there's a risk of false negatives due to missing a key move that's in flight (note that keys can't be moved atomically since they're multi-byte strings). The idea in this paper is to serialise on a single writer (thereby avoiding deadlocks), and to search forward for a valid 'cuckoo path' (series of displacements), followed by tracing that path backwards and moving keys one at a time, effectively moving the hole, not the key. Therefore there's no way the key can not be present in the table; in fact it might be in two places at once. Writer / reader conflicts might happen when a key gets moved out of the space in which the reader looks for it. The answer is to optimistically lock by using version numbers (striped across many keys, to stay cache and space efficient, nice idea), and an odd-even scheme. When a version is odd, the writer is moving the key and readers should retry their read from the start; when it is even, the key is correct as long as the version does not change during the read of the key (if it changes, they need to re-read the key). For more details, see [David Andersen's blog](http://da-data.blogspot.com/2013/03/optimistic-cuckoo-hashing-for.html). See also the [slides from NSDI](https://www.usenix.org/sites/default/files/conference/protected-files/fan_nsdi13_slides.pdf).

#### CLOCK-based LRU

Memcached spends 18-bytes for each key (!) on LRU maintenance; forward and backward pointers and a 2-byte reference counter. But why the need for strict LRU? Instead use CLOCK: 1-bit of recency that is set to 1 on every `Update()`. A 'hand' pointer moves around a circular buffer of entries. If the entry under the hand pointer has a recency value of 0, it is evicted, otherwise its recency value is set to 0 and the next item in the buffer is interrogated. The versioning scheme described above is used to coordinate when an item is being evicted (but may be concurrently read). If the eviction process decides to evict an entry, its version is incremented to indicate that it is being modified (i.e. to an odd value). Then the key is removed.

#### Notes

* Single writer obviously limits throughput for write-heavy workloads; not target of paper.
* General lesson is to pay attention to data structures; memcached wastes a surprising amount of space per key.
* Evaluation shows great improvements from using more efficient key comparator in the single-node, no concurrency case
* Also using a 1-byte hash of the key as a 'tag' provides early-out for non-matching keys, at the cost of a dependent memory read if the keys match (although you'd have that cost anyhow since the keys are too large to be cache efficient in a cuckoo hash bucket)
