---
id: 256
title: GFS Retrospective in ACM Queue
date: 2009-08-12T21:01:11+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=256
permalink: /gfs-retrospective-in-acm-queue/
categories:
  - computer science
  - Distributed systems
  - link
tags:
  - Distributed systems
  - filesystems
  - gfs
  - google
---
[This](http://queue.acm.org/detail.cfm?id=1594206) is a really great article. Sean Quinlan talks very openly and critically about the design of the Google File System given ten years of use (ten years!).

What's interesting is that the general sentiment seems to be that the concessions that GFS made for performance and simplicity (single master, loose consistency model) have turned out to probably be net bad decisions, although they probably weren't at the time.

There are scaling issues with GFS - the well known [many-small-files problem](http://www.cloudera.com/blog/2009/02/02/the-small-files-problem/) that also plagues HDFS, and a similar huge-files problem. Both increase the amount of metadata that a master has to maintain, and both therefore degrade performance due to some linear costs on metadata operations.

A replacement for GFS is in the works - a complete re-do, not just incremental updates to the design. This will involve, at least, a rethink of metadata storage and clearly a more fault-tolerant and scalable design. Multiple appenders, also a bugbear of HDFS, may be serialised through a single writer process, which would help the consistency issues.

> "Our user base has definitely migrated from being a MapReduce-based world to more of an interactive world that relies on things such as BigTable. Gmail is an obvious example of that. Videos aren't quite as bad where GFS is concerned because you get to stream data, meaning you can buffer. Still, trying to build an interactive database on top of a file system that was designed from the start to support more batch-oriented operations has certainly proved to be a pain point."

One of those great articles where every paragraph is rich with insight. Worth reading, probably twice.
