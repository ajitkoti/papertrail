---
id: 660
title: 'Exactly-once or not, atomic broadcast is still impossible in Kafka - or anywhere'
date: 2017-07-28T16:23:38+00:00
author: admin
layout: post
guid: http://the-paper-trail.org/blog/?p=660
permalink: /exactly-not-atomic-broadcast-still-impossible-kafka/
categories:
  - Uncategorized
---
## **Intro**

_**Update**: Jay responded on Twitter, which you can read [here.](https://twitter.com/jaykreps/status/891096229504966656)_

I read an [article recently by Jay Kreps](https://t.co/xrA4IROUue) about a feature for delivering messages &#8216;exactly-once&#8217; within the Kafka framework. Everyone&#8217;s excited, and for good reason. But there&#8217;s been a bit of a side story about what exactly &#8216;exactly-once&#8217; means, and what Kafka can actually do.

In the article, Jay identifies the safety and liveness properties of [atomic broadcast](https://en.wikipedia.org/wiki/Atomic_broadcast) as a pretty good definition for the set of properties that Kafka is going after with their new exactly-once feature, and then starts to address claims by naysayers that atomic broadcast is impossible.

For this note, I&#8217;m _not_ going to address whether or not exactly-once is an implementation of atomic broadcast. I also believe that exactly-once is a powerful feature that&#8217;s been impressively realised by Confluent and the Kafka community; nothing here is a criticism of that effort or the feature itself. But the article makes some claims about impossibility that are, at best, a bit shaky &#8211; and, well, impossibility&#8217;s kind of my jam. Jay posted his article with a [tweet](https://twitter.com/jaykreps/status/881563991742349313) saying he couldn&#8217;t &#8216;resist a good argument&#8217;. I&#8217;m responding in that spirit.

In particular, the article makes the claim that atomic broadcast is &#8216;solvable&#8217; (and later that consensus is as well&#8230;), which is wrong. What follows is why, and why that matters.

<blockquote class="twitter-tweet" data-width="550">
  <p lang="en" dir="ltr">
    This deserves a response: I think the conclusions are right but the imposs. arguments aren't. But it's 8pm in England and I'm in the pub. <a href="https://t.co/akmVv9rhW7">https://t.co/akmVv9rhW7</a>
  </p>

  <p>
    &mdash; Henry Robinson (@HenryR) <a href="https://twitter.com/HenryR/status/881591741966569472?ref_src=twsrc%5Etfw">July 2, 2017</a>
  </p>
</blockquote>



I have since left the pub. So let&#8217;s begin.

<!--more-->

## **You can&#8217;t solve Atomic Broadcast**

From the article:

> &#8220;So is it possible to solve Atomic Broadcast? The short answer is yes&#8221;

The long answer is no. It&#8217;s proved by Chandra and Toueg in their utterly fantastic [paper about weak failure detectors](https://www.cs.utexas.edu/~lorenzo/corsi/cs380d/papers/p225-chandra.pdf) (of which more later). In fact, the article follows up with a [reference](https://pdfs.semanticscholar.org/a8dc/564344a30fe6fd151d685f25d0e435128fa7.pdf) that fairly directly contradicts this, and it&#8217;s kind of worth quoting:

> &#8220;From a formal point-of-view, most practical systems are asynchronous because it is not possible to assume that there is an upper bound on communication delays. In spite of this, why do many practitioners still claim that their algorithm can solve agreement problems in real systems? This is because many papers do not formally address the liveness issue of the algorithm, or regard it as sufficient to consider some informal level of synchrony, captured by the assumption that “most messages are likely to reach their destination within a known delay δ” &#8230; This does not put a bound on the occurrence of timing failures, but puts a probabilistic restriction on the occurrence of such failures. **However, formally this is not enough to establish correctness**.&#8221; [emphasis mine]

A solution to the atomic broadcast problem satisfies the stated safety and liveness properties in all executions &#8211; i.e. no matter what delays are applied to message delivery or non-determinism in their receipt. If we&#8217;re talking about theory &#8211; and we are, in this article &#8211; then &#8216;solve&#8217; has this very strong meaning.

And because atomic broadcast and consensus are, from a certain perspective, [exactly the same problem](https://www.cs.utexas.edu/~lorenzo/corsi/cs380d/papers/p225-chandra.pdf) (Chandra and Toueg, again!) we can apply all the knowledge that&#8217;s been accrued regarding consensus and apply it to atomic broadcast. Specifically that it&#8217;s impossible to solve atomic broadcast in an asynchronous system where even one process may fail stop.

## **Yes but what if&#8230;**

There are some real mitigations to the FLP impossibility result. Some are mentioned in the article. None completely undermine the basic impossibility result.

**Randomization** (i.e. having access to a sequence of random coin-flips, locally) is a powerful tool. But randomization doesn&#8217;t solve consensus, it just makes the probability of a non-terminating execution extremely unlikely; and that doesn&#8217;t come with an upper bound on the length of any particular execution. See [this introductory wiki page](http://www.cs.yale.edu/homes/aspnes/pinewiki/RandomizedConsensus.html).

**Local timers** with bounded drift and skew would help since they prevent the system from being asynchronous at all &#8211; but in practice timers do drift and skew. See these [two ACM](http://queue.acm.org/detail.cfm?id=2655736) [queue articles](http://queue.acm.org/detail.cfm?id=2745385) for details.

As for **failure detectors**, it&#8217;s true that augmenting your system model with a failure detector can make consensus solvable. But you have to keep in mind that the FLP result is a \*fixed point\*. It doesn&#8217;t bend or yield to any attempt to disprove it by construction. Instead, if you think that you have solved atomic broadcast you have either done something impossible or you have changed the problem definition. Failure detectors that solve consensus are, therefore, impossible. Failure detectors are interesting \*because\* consensus is impossible &#8211; it&#8217;s surprising just how weak the properties of these failure detectors have to be to allow consensus to be solved, and therefore how hard building a failure detector actually is.

I understand the tedium of pedantry regarding abstruse theoretical matters, but it&#8217;s not productive to throw out precision in the same set of bath water.

## **Tinkerbell consensus**

The final set of claims from that initial section on impossibility is the toughest to support:

> &#8220;So if you don’t believe that consensus is possible, then you also don’t believe Kafka is possible, in which case you needn’t worry too much about the possibility of exactly-once support from Kafka!&#8221;

Hoo boy.

Here the article proposes &#8220;Tinkerbell Consensus&#8221;, which is roughly &#8220;if you believe in it hard enough, consensus can be solved&#8221;. Alas, we&#8217;ve been clapping our hands now for a really long time, and the fairy&#8217;s still dead.

I don&#8217;t believe that consensus can be solved. And yet, I use ZooKeeper daily. That&#8217;s because ZooKeeper doesn&#8217;t give a damn what I believe, but trundles along implementing a partial solution to consensus (where availability might be compromised) &#8211; happily agnostic of my opinions on the matter.

If Kafka purports to implement atomic broadcast, it too will fail to do so correctly in some execution. That&#8217;s an important property of any implementation, and one that &#8211; ideally &#8211; you would want to acknowledge and document, without suggesting that the system will do anything other than work correctly in the vast majority of executions.

## **Conclusion**

If I were to rewrite the article, I&#8217;d structure it thus: &#8220;exactly-once looks like atomic broadcast. Atomic broadcast is impossible. Here&#8217;s how exactly-once might fail, and here&#8217;s why we think you shouldn&#8217;t be worried about it.&#8221;. That&#8217;s a harder argument for users to swallow, perhaps, but it would have the benefit of not causing my impossibility spider-sense to tingle.
