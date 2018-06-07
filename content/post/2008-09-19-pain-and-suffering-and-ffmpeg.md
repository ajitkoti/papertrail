---
id: 55
title: Pain and suffering and ffmpeg
date: 2008-09-19T11:55:02+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=55
permalink: /pain-and-suffering-and-ffmpeg/
categories:
  - Note
tags:
  - encoding
  - ipod
  - problem
---
All I wanted to do was to transcode real media files from MIT OCW to iPod compatible mp4 on Linux. It shouldn&#8217;t have been this difficult. As of now, I still don&#8217;t have a satisfactory solution.

Problem 1: mplayer / mencoder read and play the stream correctly, but the mp4 files they produce when transcoding don&#8217;t work on the iPod. In particular, they&#8217;re not readable by any utilities I have such as [Easytag](http://easytag.sourceforge.net/downloads.htm) and [Amarok](http://amarok.kde.org).

Problem 2: ffmpeg can&#8217;t read rv30 files, so won&#8217;t encode them. One possibility is to use mencoder to encode to something mutually acceptable, but that involves transcoding twice which is far below ideal.

As of now, I&#8217;ve found some of the videos I want to watch on [Google Video](http://video.google.com/videoplay?docid=-2333306016564732003&ei=n4LTSKD_D5K22wLOvJC1Ag&q=mit+algorithms), but that&#8217;s not a guaranteed solution. Uploading them to Google Video just to download them again is also a non-starter.

Anyone have any ideas?