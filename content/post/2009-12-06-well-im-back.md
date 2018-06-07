---
id: 257
title: '&#8220;Well, I&#8217;m back.&#8221;'
date: 2009-12-06T22:35:42+00:00
author: Henry
layout: post
guid: http://the-paper-trail.org/blog/?p=257
permalink: /well-im-back/
categories:
  - Note
tags:
  - meta
---
Astute readers may have noticed that this blog was unavailable for the past three-and-a-half months. The short reason for this was that the computer on which Paper Trail was hosted was on a boat in the Atlantic and it is surprisingly hard to get a good internet connection out there, let alone a power supply to the shipping crate it was in.

The long reason was that I have now moved across the ocean, from Cambridge to San Francisco, where I am living in order that the commute to [Cloudera](http://www.cloudera.com/) be a little shorter than 24 hours. My possessions finally arrived on Thursday &#8211; over three months since we shipped them &#8211; and we are finally getting everything in order, including getting this blog back online. I&#8217;m now hosted at [Bluehost](http://www.bluehost.com), and have transferred all the old posts over to the new WordPress installation. I don&#8217;t yet have access to the images, so unfortunately those wil have to wait, but most other things are in order. Please excuse the dust as I find out which links are broken. 

The old address &#8211; http://hnr.dnsalias.net/wordpress &#8211; will still work, but the correct link is now our very own domain: <http://the-paper-trail.org/>. Hopefully we will start showing up in Google again when the crawlers do their thing. Please update your rss readers &#8211; those of you who are still following and haven&#8217;t deleted my feed in disgust. Does anyone even still use rss readers anymore?

I am in the process of deciding what to write about for the next few posts. I still have the remainder of the theory of computation posts to write, which would be fun to do but is a bit of a departure from the systems focus. I never really fully explained Byzantine Fault Tolerance &#8211; at least, I never got as far as describing Zyzzyva and other modern systems. At the same time, some interesting stuff in systems research has happened since the blog went quiet &#8211; Google released the [Go](http:/golang.org) programming language which is intriguing for writing user-space systems software. [SOSP 2009](http://www.sigops.org/sosp/sosp09/) happened, with some very cool papers which I really want to write about. And I&#8217;ve been busy myself &#8211; I was recently made a committer on the [Apache ZooKeeper](http://hadoop.apache.org/zookeeper) project, which is a distributed coordination system written by some engineers and researchers at Yahoo!, and is very cool. My largest contribution was a [patch](http://issues.apache.org/jira/browse/ZOOKEEPER-368) for &#8216;observers&#8217; &#8211; which are listeners, in Paxos terminology &#8211; which help maintain the read performance of the cluster as the number of clients scales. 

So, lots going on, plenty to write about, and some exciting possibilities coming down the queue. Good to be back.