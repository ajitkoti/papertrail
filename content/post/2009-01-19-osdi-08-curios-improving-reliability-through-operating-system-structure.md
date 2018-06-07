---
id: 139
title: 'OSDI &#039;08 &#8211; CuriOS: Improving Reliability Through Operating System Structure'
date: 2009-01-19T17:28:01+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=139
permalink: /osdi-08-curios-improving-reliability-through-operating-system-structure/
categories:
  - computer science
  - Operating systems
tags:
  - Operating systems
  - osdi
  - paper review
---
The second paper from OSDI that I&#8217;ll mention here is one I&#8217;ll only treat briefly &#8211; partly because it&#8217;s a bit lightweight compared to some, and partly because I&#8217;m writing in a hurry. [CuriOS: Improving Reliability Through Operating System Structure](http://www.usenix.org/events/osdi08/tech/full_papers/david/david.pdf) attacks a problem with recovery from errors in microkernel operating systems.

<!--more-->

The problem looks like this. Imagine you have a service, say a file system server, which as a client you are talking to. Suddenly it crashes. How should the service recover? The service may have lost any state that it was maintaining, which impacts clients &#8211; for example, the file system server may no longer recognise the file handles the clients present for reading. Worse, if the crash was caused by a software bug, it is possible that the server has corrupted state belonging to other clients before it crashes, resulting in incorrect behaviour and perhaps crashing the clients as a knock-on effect.

The paper identifies several mechanisms for dealing with this problem, and then notes the flaws in each. Persisting the state across a restart may not work, since as noted above the state may be corrupted before the crash occurs. Restarting the server, as mentioned above, loses client state. Checkpointing the server and client, to roll back to a known-good previous state works, but is rather expensive and requires clients to deal with going back in time. Engineering clients to deal with server crashes at any time also works, but is expensive in terms of code complexity and pushes a lot of logic for failure detection into the client.

CuriOS is an object-oriented operating system which includes a design that mitigates the problems of server crashes. The idea behind it is extremely simple. Client state is no longer stored with the server, but with the clients themselves. However, it&#8217;s mapped into a memory area that the clients cannot themselves access &#8211; the rationale being that this avoids security problems from clients being able to edit how they appear to a server. When a function is called against a server, the calling client&#8217;s state is mapped into the server&#8217;s address space &#8211; and only that client&#8217;s state. This helps prevent errors from propagating across all clients since there is no physical way a server can corrupt two client&#8217;s state at once.

That, in a nutshell, is the new contribution of the paper. It&#8217;s very simple, but a very neat idea. As the paper notes, pushing state onto the client is not new (NFS does it, for one), but isolating the state &#8211; even from the client itself &#8211; is the key step.

The rest of the paper is taken up with content including a review of how popular microkernel designs (such as Minix3, L4, Chorus and EROS) fail when services fail. One noteworthy section is the inclusion of a set of observations regarding design principles for error recovery and fault isolation in microkernels. These are: first, make sure that addresses (such as file handles) persist across server restarts to ensure transparency. Secondly prevent clients from either timing out or issuing new calls during a period of recovery to prevent errors from propagating back to the client. Thirdly, client state should persists across server restarts and finally, client state should be isolated.

The evaluation involves measuring the memory and CPU overhead of the implementation in CuriOS. CuriOS supplies &#8216;protected objects&#8217; which are services behind wrappers that take care of the mapping of client state and the restarts upon failure (signalled by C++ exceptions). CPU overhead is non-trivial &#8211; the time per call nearly doubles (but is still on the order of microseconds). The main cause of this cost is identified as the time taken to flush the TLB between switching between page tables on the switch into a protected object. Recovery time is not quantified beyond saying it&#8217;s &#8216;a few hundred microseconds&#8217;. The memory issue is similarly fudged. A lot of space is dedicated to explaining why state must at least be 1KB &#8211; it has to be rounded up in size to the nearest number of pages. However, the best figure is &#8216;on the order of tens of kilobytes&#8217; when &#8216;there are a small number of clients&#8217;. It&#8217;s not clear what small means in this context, or if it&#8217;s realistic. It&#8217;s also not clear how much of this is a consequence of the CuriOS design, and how much is just to be expected with microkernels.

The ability to recover from errors is measured, slightly strangely. Two kinds of faults are injected into servers (one for every test). Bit flips simply mutate the contents of a register. These errors may go unnoticed, but if they are detected then CuriOS restarts the server. The second type of fault is a memory error, which simply returns a kind of &#8216;bad read&#8217; code. These are always detected, and CuriOS is able to recover every time.

Although these are sound faults to inject, there&#8217;s a weird reluctance in the paper to compare the behaviour of CuriOS against existing systems. The paper simply says &#8216;all these faults would result in a service or system failure in most existing operating systems&#8217; &#8211; if true, it would have strengthened the case of CuriOS to show how many more faults it is immune to than traditional microkernels. The authors also measure the percentage of times the faults are recovered from &#8211; but do not precisely characterise what they mean by &#8216;recovered&#8217;, other than to say &#8216;useable&#8217;. Sometimes clients are disconnected &#8211; which I thought was one of the conditions they were trying to avoid &#8211; but instead they count these as successes.

There are things to like about this paper, and things to be a bit suspicious of. The evaluation feels weirdly fudged, but the idea is very nicely motivated, and this seems like a decent solution. The overhead of the context switches are high; does this impact the savings gained from a microkernel design where context switches into the kernel are in general avoided?