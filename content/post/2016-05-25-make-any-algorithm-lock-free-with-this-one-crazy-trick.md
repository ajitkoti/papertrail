---
id: 641
title: Make any algorithm lock-free with this one crazy trick
date: 2016-05-25T22:51:03+00:00
author: admin
layout: post
guid: https://the-paper-trail.org/blog/?p=641
permalink: /make-any-algorithm-lock-free-with-this-one-crazy-trick/
categories:
  - Uncategorized
---
Lock-free algorithms often operate by having several versions of a data structure in use at one time. The general pattern is that you can prepare an update to a data structure, and then use a machine primitive to atomically install the update by changing a pointer. This means that all subsequent readers will follow the pointer to its new location - for example, to a new node in a linked-list - but this pattern can’t do anything about readers that have already followed the old pointer value, and are traversing the previous version of the data structure.

<!--more-->

Those readers will see a correct, linearizable version of the data structure, so this pattern doesn’t present a correctness concern. Instead, the problem is garbage collection: who retires the old version of the data structure, to free the memory it’s taking now that it’s unreachable? To put it another way: how do you tell when all possible readers have finished reading an old version?

Of course, there are many techniques for solving this **reclamation** problem. See [this paper](http://csng.cs.toronto.edu/publication_files/0000/0159/jpdc07.pdf) for a survey, and [this paper](http://www.cs.utoronto.ca/~tabrown/debra/paper.pdf) for a recent improvement over epoch-based reclamation. RCU, which is an API for enabling single-writer, multi-reader concurrency in the Linux kernel, has an elegant way of solving the problem.

Every reader in RCU marks a critical section by calling `rcu_read_lock()` / `rcu_read_unlock()`. Writers typically take responsibility for memory reclamation, which means they have to wait for all in-flight critical sections to complete. The way this is done is, conceptually, really really simple: during a critical section, a thread may not block, or be pre-empted by the scheduler. So as soon as a thread yields its CPU, it’s guaranteed to be out of its critical section.

This gives RCU a really simple scheme to check for a grace period after a write has finished: try to run a thread on every CPU in the system. Once the thread runs, a context switch has happened which means any previous critical sections must have completed, and so any previous version of the data structure can be reclaimed. The readers don't have to do anything to signal that they are finished with their critical section: it's implicit in them starting to accept context switches again!

In reality, this is not quite what RCU does (but the idea is the same, see [this terrific series](https://lwn.net/Articles/262464/)). Instead, it takes advantage of kernel context-switch counters and waits for them to increase:

> “In practice Linux implements synchronize\_rcu by waiting for all CPUs in the system to pass through a context switch, instead of scheduling a thread on each CPU. This design optimizes the Linux RCU implementation for low-cost RCU critical sections, but at the cost of delaying synchronize\_rcu callers longer than necessary. In principle, a writer waiting for a particular reader need only wait for that reader to complete an RCU critical section. The reader, however, must communicate to the writer that the RCU critical section is complete. The Linux RCU implementation essentially batches reader-to-writer communication by waiting for context switches. When possible, writers can use an asynchronous version of synchronize\_rcu, call\_rcu, that will asynchronously invokes a specified callback after all CPUs have passed through at least one context switch.”

From [RCU Usage In the Linux Kernel: One Decade Later](http://www2.rdrop.com/~paulmck/techreports/RCUUsage.2013.02.24a.pdf).

The most elegant thing about vanilla RCU is that the system is lock-free by definition not by design - it has nothing to do with the semantics of RCU’s primitives, and everything to do with the fact that being in the kernel allows you to enforce a sympathetic system model. If threads can’t block or otherwise be prevented from making progress, any (non-pathological) algorithm must, by definition, always make progress! Even a concurrency scheme that nominally used spinlocks to protect critical sections would be lock-free, because every thread would exit their critical section in bounded time - the other threads would all be serialised behind this lock, but there would be progress.

(There are other flavours of RCU that don’t restrict critical sections in this way, as they require critical sections to allow pre-emption).
