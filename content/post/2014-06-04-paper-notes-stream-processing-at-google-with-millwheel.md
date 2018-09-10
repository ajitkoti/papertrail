---
id: 582
title: 'Paper notes: Stream Processing at Google with Millwheel'
date: 2014-06-04T12:07:04+00:00
author: Henry
layout: post
guid: https://the-paper-trail.org/blog/?p=582
permalink: /paper-notes-stream-processing-at-google-with-millwheel/
categories:
  - Paper notes
---

### Millwheel: Fault-Tolerant Stream Processing at Internet Scale

  _Akidau et. al., VLDB 2013_

#### The big idea

Streaming computations at scale are nothing new. Millwheel is a standard DAG stream processor, but
one that runs at 'Google' scale. This paper really answers the following questions: what guarantees
should be made about delivery and fault-tolerance to support most common use cases cheaply? What
optimisations become available if you choose these guarantees carefully? <!--more-->

#### Notable features

* A tight bound on the timestamp of all events still in the system, called the **low
  watermark**. This substitutes for in-order delivery by allowing processors to know when a time
  period has been fully processed (nice observation: you don't necessarily care about receiving
  events in order, but you do care about knowing whether you've seen all the events before a certain
  time).

* Persistent storage available at all nodes in the stream graph

* Exactly once delivery semantics.

* Triggerable computations called **timers** that are executed in node context when either a low
  watermark or wall clock time is observed. This can be modelled as a separate source of null events
  that allows periodic roll-up to be done.Each node in the graph receives input events, optionally
  computes some aggregate, and also optionally emits one or more output events as a result. In order
  to scale and do load balancing, each input record must have a key (the key extraction function is
  user defined and can change between nodes); events with identical keys are sent to the same node
  for processing.

* The **low watermark at A** is defined as <tt>min(oldest event still in A, low watermark amongst
  all streams sending to A)</tt>. Watermarks are tracked centrally. Note that the monotonic
  increasing property of watermarks requires that they enter the system in time order, and therefore
  we can't track arbitrary functions as watermarks. State is persisted centrally. To ensure
  atomicity, only one bulk write is permitted per event. To avoid zombie writers (where work has
  been moved elsewhere through failure detection or through load balancing), every writer has a
  lease or sequencer that ensures only they may write. The storage system allows for this to be
  atomically checked at the same time as a write (i.e. conditional atomic writes). Emitted records
  are checkpointed before delivery so that if an acknowledgment is not received the record can be
  re-sent (duplicates are discarded by MillWheel at the recipient). The checkpoints allow
  fault-tolerance: if a processor crashes and is restarted somewhere else any intermediate
  computations can be recovered. When a delivery is ACKed the checkpoints can be garbage collected.

* The Checkpoint->Delivery->ACK->GC sequence is called a **strong production**. When a processor
  restarts, unacked productions are resent. The recipient de-duplicates. In some cases, Millwheel
  users can optimise by allowing events to be sent before the checkpoint is committed to persistent
  storage - this is called a **weak production**. Weak productions are possible usually if the
  processing of an event is idempotent with respect to the persistent storage and event production
  (i.e. you always send the same event once you commit to sending it once (so aggregates over time
  don't necessarily work) and / or receipt of those events multiple times doesn't cause
  inconsistencies).
