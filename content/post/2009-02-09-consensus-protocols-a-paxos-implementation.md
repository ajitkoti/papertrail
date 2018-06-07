---
id: 190
title: 'Consensus Protocols: A Paxos Implementation'
date: 2009-02-09T19:37:44+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=190
permalink: /consensus-protocols-a-paxos-implementation/
wp-syntax-cache-content:
  - |
    a:1:{i:1;s:3332:"
    <div class="wp_syntax" style="position:relative;"><table><tr><td class="code"><pre class="python" style="font-family:monospace;"><span style="color: #ff7700;font-weight:bold;">if</span> message <span style="color: #66cc66;">==</span> <span style="color: #008000;">None</span>:
    <span style="color: #808080; font-style: italic;"># Only run every 15s otherwise you run</span>
    <span style="color: #808080; font-style: italic;"># the risk of cutting good protocols off in their prime</span>
    <span style="color: #ff7700;font-weight:bold;">if</span> <span style="color: #008000;">self</span>.<span style="color: black;">isPrimary</span> <span style="color: #ff7700;font-weight:bold;">and</span> <span style="color: #dc143c;">time</span>.<span style="color: #dc143c;">time</span><span style="color: black;">&#40;</span> <span style="color: black;">&#41;</span> - <span style="color: #008000;">self</span>.<span style="color: black;">lasttime</span> &amp;gt<span style="color: #66cc66;">;</span> <span style="color: #ff4500;">15.0</span>:
    <span style="color: #808080; font-style: italic;"># if no message is received, we take</span>
    <span style="color: #808080; font-style: italic;"># the chance to do a little cleanup</span>
    <span style="color: #ff7700;font-weight:bold;">for</span> i <span style="color: #ff7700;font-weight:bold;">in</span> <span style="color: #008000;">xrange</span><span style="color: black;">&#40;</span><span style="color: #ff4500;">1</span><span style="color: #66cc66;">,</span><span style="color: #008000;">self</span>.<span style="color: black;">highestInstance</span><span style="color: black;">&#41;</span>:
    <span style="color: #ff7700;font-weight:bold;">if</span> <span style="color: #008000;">self</span>.<span style="color: black;">getInstanceValue</span><span style="color: black;">&#40;</span> i <span style="color: black;">&#41;</span> <span style="color: #66cc66;">==</span> <span style="color: #008000;">None</span>:
    <span style="color: #ff7700;font-weight:bold;">print</span> <span style="color: #483d8b;">&quot;Filling in gap&quot;</span><span style="color: #66cc66;">,</span> i
    <span style="color: #008000;">self</span>.<span style="color: black;">newProposal</span><span style="color: black;">&#40;</span> <span style="color: #ff4500;">0</span><span style="color: #66cc66;">,</span> i <span style="color: black;">&#41;</span>
    <span style="color: #008000;">self</span>.<span style="color: black;">lasttime</span> <span style="color: #66cc66;">=</span> <span style="color: #dc143c;">time</span>.<span style="color: #dc143c;">time</span><span style="color: black;">&#40;</span> <span style="color: black;">&#41;</span></pre></td></tr></table><p class="theCode" style="display:none;">if message == None:
    # Only run every 15s otherwise you run
    # the risk of cutting good protocols off in their prime
    if self.isPrimary and time.time( ) - self.lasttime &amp;gt; 15.0:
    # if no message is received, we take
    # the chance to do a little cleanup
    for i in xrange(1,self.highestInstance):
    if self.getInstanceValue( i ) == None:
    print &quot;Filling in gap&quot;, i
    self.newProposal( 0, i )
    self.lasttime = time.time( )</p></div>
    ";}
categories:
  - Distributed systems
tags:
  - consensus
  - Distributed systems
  - paxos
---
It's one thing to wax lyrical about an algorithm or protocol having simply read the paper it appeared in. It's another to have actually taken the time to build an implementation. There are many slips twixt hand and mouth, and the little details that you've abstracted away at the point of reading come back to bite you hard at the point of writing.

I'm a big fan of building things to understand them - this blog is essentially an expression of that idea, as the act of constructing an explanation of something helps me understand it better. Still, I felt that in order to be properly useful, this blog probably needed more code.

So when, yesterday, it was suggested I back up my previous post on Paxos with a toy implementation I had plenty of motivation to pick up the gauntlet. However, I'm super-pressed for time at the moment while I write my PhD thesis, so I gave myself a deadline of a few hours, just to keep it interesting.

A few hours later, I'd written [this](https://github.com/henryr/toy_paxos) from-scratch implementation of Paxos. There's enough interesting stuff in it, I think, to warrant this post on how it works. Hopefully some of you will find it useful, and something you can use as a springboard to your own implementations. You can run an example by simply invoking <tt>python toy_paxos.py</tt>.

<!--more-->

I'm not going to go over again how Paxos works - in fact I'm going to assume that you've read and understood my [previous post on the subject](http://hnr.dnsalias.net/wordpress/?p=173). However, there's an extra couple of bits of detail that you'll need to understand some of the things in the code.

It might help to clarify the abstraction this code is trying to present. I'm using Paxos to simulate a _state machine_, where the leader proposes a value for the next command for the acceptors to agree upon. Therefore, rather than run just one instance of Paxos, we're running many consecutive instances, the  \\(n^{th}\\) of which is concerned with agreeing on the  \\(n^{th}\\) command in sequence. In the code, this index is identified as the <tt>instanceID</tt>. So now every proposal carries with it a value, a proposal ID and an instance ID.

The idea is to agree upon an ordered _history_ of state machine commands, one command per instance. Clients propose a value for a command they'd like to execute. The consensus service orders these commands and notifies any interested parties. Occasionally, it might happen that there are holes in the history that a leader knows about. In that case, the leader can simply propose a 'no-op' value (here 0), and will learn about the real value if there is one, as Paxos ensures that a previously accepted value will stay accepted. Otherwise, 0 gets inserted in the history with no ill-effect.

The code is structured into four main classes. Two are 'actor' classes, <tt>PaxosLeader</tt> and <tt>PaxosAcceptor</tt>. These two classes receive messages from each other and the outside world (in the leader's case), kick off protocol instances, get notified when protocols have finished and so on. The other two important classes are <tt>PaxosLeaderProtocol</tt> and <tt>PaxosAcceptorProtocol</tt>. These are state-machine classes that keep track of one actor's view of the state of a single protocol instance. The important code for these two is mainly in <tt>doTransition</tt>, which takes a message object, looks at the current protocol state, and figures out what to do with it.

Of the two actor classes, <tt>PaxosAcceptor</tt> is much the simpler. Both classes are attached to a <tt>MessagePump</tt> class, which listens on a socket and copies messages to a queue for the actor class to consume one by one.

(Note the convoluted thread-within-thread copying from the socket onto a synchronised queue inside the <tt>MessagePump</tt> - the problem is that I was filling the UDP receive buffer quicker than the actor could pull messages from the socket, so I needed to empty the buffer as soon as possible, asynchronously. I don't actually know if TCP would have helped here - does the acknowledgement of a packet get sent when the socket is read?)

<tt>MessagePump</tt> calls back into <tt>recvMessage</tt>, which in the <tt>PaxosAcceptor</tt> case simply checks if the message contains a new proposal. If it does, a new <tt>PaxosAcceptorProtocol</tt> object is created and set loose. Otherwise, the corresponding protocol is looked up through the <tt>InstanceRecord</tt> structure (of which there is one per instance, and simply contains a table of all known proposals for that instance), and <tt>doTransition</tt> is called through.

For <tt>PaxosLeader</tt>, <tt>recvMessage</tt> is a bit more complicated. The first chunk of code:

<pre lang="python">if message == None:
    # Only run every 15s otherwise you run
    # the risk of cutting good protocols off in their prime
    if self.isPrimary and time.time( ) - self.lasttime &gt; 15.0:
        # if no message is received, we take
        # the chance to do a little cleanup
        for i in xrange(1,self.highestInstance):
            if self.getInstanceValue( i ) == None:
                print "Filling in gap", i
                self.newProposal( 0, i )
        self.lasttime = time.time( )</pre>

only gets run if there is no message to process (and only then if it's been more than 15s since we last ran this code). The idea is to run through all the instances we know about, and check if we've got a value for them. If we haven't then we propose 0 as a no-op value, as above.

Otherwise, we look at the type of message we've received. Heartbeat messages come from other leaders, and are used to indicate that a leader thinks it is the current primary. If a leader thinks it is the primary and receives a heartbeat from a leader whose port number is higher, it yields to that leader. However, if no heartbeats have been heard for a period of time, any leader can assert itself as the primary, and start issuing heartbeats.

If the message comes from a client, the leader constructs a new protocol object and passes control to it, after saving it in the <tt>InstanceRecord</tt> for the current instance. This is done in <tt>newProposal</tt>. The current instance number is defaulted to one larger than the previous proposal. However, if we are filling in gaps then we may want to force the instance to an earlier value.

If the message belongs to a protocol we know about, we call through the <tt>doTransition</tt> as before. However, leaders also want to know about accepted proposals so that, if they need to take over, they can be reasonably up-to-date. Therefore every leader, whether or not it is the primary, runs the leader protocol for any proposals that have got to the accept phase - that is, they listen for a quorate number of accepts and then are aware when a proposal has been accepted and at what value.

Some other niceties: you can substitute <tt>AdversarialMessagePump</tt> in to arbitrarily delay messages and simulate asynchrony. Acceptors can simulate failure by calling <tt>fail( )</tt> on them - they then stop responding to messages. In a proper implementation, you'd want to log to persistent storage every state change in both leader and acceptor so that if they were to genuinely crash and fail they could recover even having lost their memory.

This toy implementation has been designed to run on a single machine in one Python process. Therefore every message pump binds to localhost on a unique port (it would not be hard at all to generalise this to use proper IP addresses). There are doubtless a few bugs, and more than a few bits of ugliness. Let me know what you find!
