---
id: 427
title: On some subtleties of Paxos
date: 2012-11-03T19:02:22+00:00
author: Henry
layout: post
guid: http://the-paper-trail.org/blog/?p=427
permalink: /on-some-subtleties-of-paxos/
categories:
  - Distributed systems
---
There&#8217;s one particular aspect of the Paxos protocol that gives readers of this blog &#8211; and for some time, me! &#8211; some difficulty. This short post tries to clear up some confusion on a part of the protocol that is poorly explained in pretty much every major description.
  
<!--more-->


  
This is the observation that causes problems: two different nodes can validly accept different proposals for the same Paxos instance. That means that we could have a situation where node  <tt>A</tt> has accepted a proposal  <tt>P=(S, V)</tt>, and node  <tt>E</tt> has accepted a proposal  <tt>P'=(S', V')</tt>. We can get there very easily by having two competing proposers that interleave their prepare phases, and crash after sending an accept to one node each. On the face of it, this is a very concerning situation &#8211; how can two nodes both believe a different value has been accepted? Doesn&#8217;t this violate one of the consensus guarantees of uniformity?

The answer lies in the fact that the nodes doing the accepting are not (necessarily) the nodes that &#8216;learn&#8217; about the agreed value. Paxos calls for a distinguished category of nodes, called &#8216;learners&#8217;, which hear about acceptance events from the nodes doing the accepting. We call the latter nodes &#8216;acceptors&#8217;, and say that learners &#8216;commit&#8217; to a value once they can be sure it&#8217;s never going to change for a given Paxos instance.

But when does a learner know that a value has really been accepted? It can&#8217;t just go on the first acceptance that it receives (since as we have shown, two different acceptors can have accepted different values and may race to send their values to the learners). Instead, a learner must wait for a majority of acceptors to return the same proposal. Once  <tt>N/2 + 1</tt> acceptors are in agreement, the learner can commit the value to its log, or whatever is required once consensus is reached. The rest of this post shows why this is both necessary and sufficient.

If no majority of acceptors have accepted the same value, it&#8217;s trivial to see why a learner cannot commit to a value sent by any acceptor, for the same race-based argument made earlier. A more interesting case is the following: suppose that a majority of acceptors have accepted a proposal with the same value, but with different sequence numbers (i.e. proposed by a different proposer). Can a learner commit to that value once it has learnt about all the acceptances? In the following section we show that it can. 

### Conditions for learner commit

**Theorem:** _Let a majority of acceptors have accepted some proposal with value  <tt>V</tt>, and let  <tt>P = (S,V)</tt> be the proposal with the largest sequence number  <tt>S</tt> amongst all those acceptors. Then there is no proposal  <tt>P' = (S', V')</tt> that is accepted at any node with  <tt>S'>S</tt> and  <tt>V' != V</tt>._

**Proof:** We proceed in two steps. The first shows that when <tt>P</tt> is accepted, there is no proposal already accepted at any node with a later sequence number. Assume that this is false, and some node has accepted <tt>P'</tt>. Then a majority of acceptors must have promised to accept only proposals with sequence number <tt>S'' > S'</tt>. Since <tt>S < S'</tt>,  <tt>P</tt> cannot be accepted by a majority of acceptors, contradicting the assumption. 

Second, we prove by a very similar argument that once  <tt>P</tt> has been accepted, no node will accept a proposal like  <tt>P'</tt>. Again, assume this is false. Then a majority of acceptors must have sent a <tt>promise</tt> of  <tt>(S', V')</tt> in order for the proposer of  <tt>P'</tt> to be sending <tt>accept</tt> messages. If so, then either that same majority should have ignored the <tt>accept</tt> message for  <tt>P</tt> (since  <tt>S < S'</tt>), or  <tt>V' = V</tt> if the <tt>accept</tt> of <tt>P</tt> happened before the proposal of <tt>P</tt> (by the first half of this proof we know that there is no proposal with a later sequence number than  <tt>S</tt> already accepted, so the proposer is guaranteed to choose  <tt>V</tt> as the already accepted value with the largest sequence number). In either case there is a contradiction;  <tt>P</tt> has not been accepted or  <tt>V = V'</tt>. 

What this theorem shows is that once a value has been accepted by a majority of acceptors, no proposal can change it. The sequence number might change (consider what happens if a new proposer comes along and runs another proposal over the same instance - the sequence number will increase at some acceptors, but the proposer must choose the majority value for its accept message). But since the majority-accepted value will never change, the learners can commit a value when they hear it from a majority of acceptors.

### Fault tolerance

Now it's instructive to think about what this means for Paxos' fault-tolerance guarantees. Imagine that a proposal was accepted at a minimal (i.e.  <tt>N/2+1</tt> nodes) majority of acceptors before the proposer crashed. In order for the value to be committed by a learner, every one of those acceptors must successfully send its accepted value on to the learners. So if a single node in that majority fails, that Paxos instance will not terminate for all learners. That appears to be not as fault-tolerant as we were promised.

There are several ways to interpret this fact. The first is that Paxos only guarantees that it will continue to be correct, and live, with up to  <tt>N/2 + 1</tt> failures; and _failing to reach agreement for a single proposal does not contravene these properties_. It's also true that if the proposer dies before sending any accept messages, that proposal will also never complete. However, another proposer can always come along and finish that instance of the protocol; it's this that is no longer true if a majority of acceptors fail. 

The second interpretation is that it makes sense for acceptors to also act as learners, so that they can update their values for a given Paxos instance once they realise that consensus is complete. It's often true that learners and acceptors are the same thing in a real Paxos deployment, and the aim is usually to have as many acceptors up-to-date as possible. 

So that's a short look at how the distribution of accepted proposals can evolve during Paxos, and how the protocol guarantees that eventually the cluster will converge on a value that will never change.