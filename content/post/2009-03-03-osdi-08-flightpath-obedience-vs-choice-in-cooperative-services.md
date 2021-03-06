---
id: 205
title: "OSDI '08: FlightPath: Obedience vs. Choice in Cooperative Services"
date: 2009-03-03T15:45:18+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=205
permalink: /osdi-08-flightpath-obedience-vs-choice-in-cooperative-services/
categories:
  - computer science
  - Distributed systems
  - Paper Walkthrough
tags:
  - Distributed systems
  - osdi
  - paper review
  - peer-to-peer
---
This is one of my favourite papers from OSDI '08 (yes, still doing a few reviews, trying to get to five or so before SOSP...). [FlightPath](http://www.usenix.org/events/osdi08/tech/full_papers/li_h/li_h.pdf) is a system developed by some folks mainly at UT Austin for peer-to-peer streaming in dynamic networks. This is a reasonably challenging problem in itself, although one that's seen a good deal of work before. However, the really cool thing about this paper is that they treat participants in the network as potentially rational agents. Since Lamport's seminal work on the Byzantine generals problem, it's been standard practice to assign one of two behaviour modes to members of distributed systems: either you're alturistic, which means that you do exactly what the protocol tells you to do, no matter what the cost to yourself, or Byzantine, which means that you do whatever you like, again no matter what the cost to yourself.

It was realised recently that this is a false dichotomy: there's a whole class of behaviour that's not captured by these two extremes. _Rational_ agents participate in a protocol as long as it is worth their while to do so. At its most simple, this means that rational agents will not incur a cost unless they expect to recoup a benefit that is worth equal to or more than the original cost to them. This gave rise to the Byzantine-Alturistic-Rational (BAR) model, due to the same UTA group, which can be used to more realistically model the performance of peer-to-peer protocols.

<!--more-->



Protocols that involve rational agents must provide _incentives_ for agents to participate. The paper gives the example of KaZaA, a p2p file-sharing service which required users to download and display adverts in return for access to the network. However, once users reverse-engineered the protocol they were able to develop a client that didn't need to display any adverts. There was no longer any incentive for them to do so (since they had access to the network anyhow), and so it was no longer rational for them to incur the cost (measured, probably, in annoyance) of displaying the adverts.

The paper blames this on a lack of formal analysis of the protocol, using long established game theoretic tools such as Nash equilibria. BitTorrent falls foul of the same accusation, as there are ways to cheat the protocol and freeride without having to provide your fair share of upstream bandwidth.

Contrasting against the informal, but clearly practical (judging by their popularity), systems are those that are rigorously specified and analysed, but impractical due to a lack of flexibility afforded by the need to make strong assumptions. Previous work in the BAR model does not allow for dynamic membership, for example. This drawback is the main hurdle that the authors overcome in this paper. By slightly weakening their model of rational behaviour the authors can develop a protocol that is dynamically adaptable to changing network conditions.

This paper proposes FlightPath, a peer-to-peer content streaming algorithm for dynamic networks. The targeted domain is streaming video, where a source produces packets at some fixed rate which are then distributed amongst the peers via a heavily modified gossip protocol. The paper splits presentation of this protocol into two parts. The first introduces a basic protocol that doesn't display good performance, but has the basic mechanisms in place for curbing Byzantine behaviour and providing suitable incentives. The second part gives a number of added features which make the protocol more practical.

One of the significant technical challenges with streaming video is finding enough bandwidth. Even if you've got a server sitting on a very wide network connection, you're still likely to use all of it as the number of clients gets large. A P2P protocol makes a lot of sense in this light, because each client can donate its own upload bandwidth to the effort and retransmit the stream to peers that can't get in on the server. However, this then relies on every node participating fairly in the protocol: the scalability of the network relies upon peers contributing back some resources to compensate for those that they use.

There are three main types of actor in the FlightPath protocol. The _tracker_, much like a BitTorrent tracker, keeps a view on the current membership of the network. The tracker is assumed to behave properly, and thus can be used to evict misbehaving peers from the network. Streams eminate from a _source_ at the rate of _num_ups_ per round, where a round is some number of seconds long. Therefore a peer should deliver _num_ups_ updates per round; if it does not a round is considered _jittered_. The goal of FlightPath is to minimise jittered rounds.

## Trades

The basic protocol is built around the idea of _trades_, which are the units of exchange that make up a gossip protocol. Different peers are likely to have received two different sets of frames, so they promise to give each other the frames that the other is missing in return for the same from their peer. Peers incur a cost in participating in an exchange - upload bandwidth is a limited resource and if a peer could get away without actually uploading its promised frames it would be rational to do so, not least because the same bandwidth could be promised to another peer for further gain.

Therefore the trade protocol needs to be set up in such a way as to incentivise both parties to send the frames they promise, and also to allow peers to prove that they have been cheated if that should happen. The protocol allows for the exclusion of misbehaving peers, so that eventually the cost to them is maximal as they won't receive anything at all. This requires identification, and a non-forgeable proof of misbehaviour which a peer can present to the tracker to evict the misbehaving peer from the network.

The trade protocol has three phases, In the first, the initiating peer asks another peer for its frame history, and upon receiving it replies with its own. The second phase has both peers send each other a 'briefcase' containing the frames they found to be missing from the frame histories they received. Each briefcase is encrypted so that it's of no utility to the receiving peer without the decryption key. Also sent with the briefcase is a 'promise' that is unforgeably signed by the sending peer. A promise contains a declaration of the frames that a peer is sending. The promise commits a peer to good behaviour, as if it can be shown that a peer does not send the frames it has promised then the peer will be evicted.

The final phase is entered only when a briefcase and promise has been sent and received by both peers. By this point both peers have incurred a significant cost in participating in the protocol, since the briefcase will have cost upload bandwidth. However, no utility can be realised until the keys are received, which happens in the third protocol phase. Therefore there is an incentive for both peers to send the briefcase. Once the briefcase has been sent, it costs relatively very little to transmit the briefcase key, so there's no point witholding the key.

If a peer lies about the frames it has, in order to con another peer into a tit-for-tat exchange, this will be exposed later in the protocol when the decrypted briefcase doesn't match the promise. There's no way to trick a peer into sending frames without getting any in return without incurring a cost, as downloading a briefcase costs download bandwidth, and the keys to decrypt the briefcase won't be received until a corresponding briefcase is sent.

So there is no obvious way for a rational peer to profit greatly from this protocol without ultimately incurring the ultimate sanction of being excluded from the network. If we assume, reasonably, that there's an implicit incentive in receiving the stream then it is rational to complete a trade according to the letter of the protocol.

Clearly, Byzantine peers don't behave rationally, so there is no way of incentivising them to behave. Instead, the protocol can only try to mitigate the impact of Byzantine peers as a function of the fraction of peers that are Byzantine.

## Protocol Improvements

The paper goes into some detail about the improvements to the basic protocol. It is clear from the paper that the basic protocol is flawed and impractical, and therefore that these improvements are not just feature tweaks but necessary design improvements to make the protocol useable in practice. The paper is nicely detailed on all of these, so I'm only going to quickly survey one of the interesting ones rather than regurgitate someone else's work entirely!

### Reservations

There's a wide variability in the number of possible trade partners from round to round. In order to prevent some nodes getting starved where others get overtaxed, FlightPath requires nodes to reserve a slot for trading with a peer in the previous round. This request is only granted if the receiving peer has a free slot. This curbs the number of trades that a peer will be involved with.

Further, to avoid a peer constantly trading with one specific peer, the membership of the network is divided by the tracker into bins of size  \\(\log n\\) and peers pick one bin at a time verifiably at random (using a predictable pseudorandom number generator). Further, each peer gets a custom view of the membership of each bin such that every view is expected to contain at leasdt one non-Byzantine peer (given a known fraction  \\(F_{byz}\\) of Byzantine peers).

## Churn and Network Startup

FlightPath is targeted at dynamic networks, where peers join and leave regularly. One problem with dynamic membership is that there a number of peers who are not able to contribute to the network, but need to consume resources in order to have something to trade. The tracker distributes membership updates in epochs, so a new peer would have to wait until the next epoch in order to be allowed to trade.

To get around this, FlightPath organises newly joined peers into 'tubs', such that the first tub contains live peers, and subsequent tubs are composed of newly arrived peers. Peers may select members from their own tub, and with geometrically decreasing probability from any earlier tubs. This ensures that peers get started quickly, but do not overwhelm the network. Some frames will leak into later tubs from the first tub, but then it is more likely that peers will trade these frames amongst themselves rather than pull them from the first tub.

## Wrapping Up

Evaluation is extensive, and includes measurement of the effect of faulty peers (hurrah!). FlightPath is tolerant of up to 10% of peers becoming Byzantine, although the authors only claim to have studied Byzantine behaviour that 'mappeared to cause the greatest harm'. It surprises me that there's not a model of worst-cast Byzantine behaviour that we can apply to new protocols.

The authors also quickly analyse the game-theoretic properties of FlightPath. If rational nodes are subject to an \\(\epsilon\\) -equilibrium, where they only deviate from protocol behaviour if their increase in utility is larger by a factor of at least \\(1+\epsilon\\) , the authors are able to show that FlightPath is at least a \\(1/10\\) -equilibrium if the peer values the receipt of the jitter-free stream at least 3.36 times as much as the cost of the bandwidth used to participate in the system.

I don't know much about game theory, but I like this analysis. These things always seem to come down to getting the cost function correct, so eventually you just have to appeal to what seems a reasonable valuation on the part of the user. \\(\epsilon\\) -equilibria seem a natural tool to use - people only seem to behave rationally if the return significantly outweighs the effort. For example, how many people download Ad-block for Firefox? 🙂
