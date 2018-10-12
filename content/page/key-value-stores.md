---
title: "Key-value store research"
date: 2018-06-07T14:49:07-07:00
draft: true
aliases:
  - /key-value-stores/
---
<span class="center-table">

Table of contents    |
---------------------|
[Part 0: Introduction](/key-value-stores) |
[Part 1: Masstree](/post/masstree-paper-notes/)     |
[Part 2: MICA](/post/mica-paper-notes)         |

</span>
# Introduction

Arguably, no capability is more critical to systems software than the ability to efficiently lookup
a value based on a key. Such a general primitive has a wide variety of uses, and with that comes an
equal variety in the data structures and algorithms used.

Key-value stores naturally have many, many applications. Chief amongst them from a database
perspective might be _indexing_ - storing the location of data stored in-memory or on disk.

More variety means more scope for research, and the last ten years of systems research has yielded

**CPU efficiency**: How do you make efficient use of multiple cores? How do you make sure the data
that a core needs is close to it (cache efficiency, not just cache conflicts)? Locking vs lock-free
(or a hybrid). Is it possible to exploit SIMD - or to reduce the amount of work a CPU has to do in
total?

**Concurrency**: Closely related to multicore support - how do we ensure safe but efficient
concurrent access to data structures? Locking vs lock-free (or a hybrid). Do you split work between
cores, or have them all able to do all work?

**Data considerations:** Common prefixes, skew, length, fixed-length or variable

**Storage:** In-memory, spinning disk, flash

**Interface:** Point lookup? Range query? Test and set? Transactions?

**Distribution:** Do we care about more than one node? If so, how much coordination is needed?
