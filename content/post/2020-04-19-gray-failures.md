---
title: "Gray Failures"
date: 2020-04-18T22:04:31-07:00
author: Henry
permalink: /paper-notes-gray-failures
aliases: [/gray-failures/]
draft: false
layout: post
categories:
    - Paper notes
---

_Huang et. al., HotOS 2017_ ["Gray Failure: The Achilles Heel of Cloud-Scale Systems"](https://www.microsoft.com/en-us/research/wp-content/uploads/2017/06/paper-1.pdf)

Detecting faults in a large system is a surprisingly hard problem. First you have to decide what kind of thing you want to measure, or 'observe'. Then you have to decide what pattern in that observation constitutes a sufficiently worrying situation (or 'failure') to require mitigation. Then you have to decide how to mitigate it!

Complicating this already difficult issue is the fact that the health of your system is in part a matter of perspective. Your service might be working wonderfully from inside your datacenter, where your probes are run, but all of that means nothing to your users who have been trying to get their RPCs through an overwhelmed firewall for the last hour.

That gap, between what your failure detectors observe, and what clients observe, is the subject of this paper on 'Gray Failures', which are the failure modes that happen when clients perceive an issue that is not yet detected by your internal systems. This is a good name for an old phenomenon (every failure detector I have built includes client-side mitigations to work around this exact issue).

<!--more-->

The simplest Gray Failure is given by the paper: a service runs two threads. One serves requests, the other serves heartbeats (you might do heartbeats on a separate thread so as to avoid interfering with real traffic). The request thread gets stuck, maybe due to a bug. The heartbeat thread keeps going. Your service appears available to the heartbeat failure detector, and completely unavailable everywhere else.

The paper describes some experience with Azure, although I found some of the points raised to not be very specific to Gray Failures. Take, for example, the first example where redundancy (here in the form of extra switches in a [Clos network](https://en.wikipedia.org/wiki/Clos_network)) can hurt availability and reliability, because extra hardware means the likelihood of faults increases proportionally, and so does the chance of system degradation. This is of course true, even if you were able to detect and react to all faults - no fault mitigation strategy is zero cost. It is perhaps true that the cost of these failures is higher for GF because they go unmitigated for some time.

Similarly the example of a failing data storage node that was running out of capacity seems more general: the system detected a crash fault caused by the undetected capacity crunch and eventually removed the node from the system; this leads to higher load on other nodes and eventually a cascading failure. But that would be true even if the fault had been perfectly detected.

The issue here is that *mitigation* is inappropriate, and that issue is only lightly touched on by the paper. Detection is one half of the problem, but building a system that takes complex actions to mitigate subtle faults is extremely risky. I suspect that I would let a data storage node whose disk was full crash as well, rather than trying to automate some workflow where a new disk was added and the old data was migrated (the capacity issue should be addressed by the replication protocol). Figuring out how to mitigate if, for example, your network link is flakey seems to me like a most interesting open research problem.

Gray Failures do exist, and figuring out what to do about them is challenging. The authors suggest expanding the set of measurements taken to form a richer, multi-dimensional input to the failure detector, but that seems to misrepresent the fact that standard monitoring practice is to measure a pretty high-dimensional set of metrics from every node and service. The problem is that we practitioners have poor tools for classifying that input as 'failing' or 'not failing' (think of how hard it is to decide when or if you should alert on high CPU usage across your fleet; then try doing that with 50 input metrics at once). That tooling would be a valuable research project.

Of course, the best signal to capture is exactly what your client is seeing, and the authors reasonably argue that that is completely impractical. Building approximations is always worthwhile, and there is a school of thought that any signals that are not customer facing may not be worth alerting on or mitigating. At least as a guiding principle that rather focuses the mind and prevents too much alert fatigue. (The counter argument is that you wish to predict customer issues rather than react to them, which is clearly better but very hard to do! Again, tools that understand the predictive power of observations would be valuable).

Even if you can't capture the complete client perspective, my view is that you must build fault mitigation into your clients - for simple node health detection this might include locally keeping lists of failed nodes, retries, and so on. These mitigations go a long way towards routing around failures, which can serve you well until you can actually resolve the underlying issue and return full capacity to the service.
