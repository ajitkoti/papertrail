---
id: 41
title: VirtualBox is fantastic
date: 2008-02-07T15:14:39+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=42
permalink: /virtualbox-is-fantastic/
categories:
  - Uncategorized
---
<p class="entry-body">
  For the longest time, I’ve been struggling with Xen on my Fedora desktop. Essentially, the problems are twofold: one, there’s no good solution to a broken NVidia kernel module for a Xen kernel, and I like my graphical bling, and two, Fedora doesn’t really seem to play nicely with Xen without a lot of configuration, especially if you want to run Windows XP, which I do.
</p>

The end of my particular tether came when trying to get networking to start at all in order to get at the domain to see what was wrong, and every time I tried to start the bridge all my physical network connections were turned off. Looking at the code I think this was something to do with me having <tt>eth1</tt> as my interface rather than <tt>eth0</tt>, but configuring Xen is hard enough without having to fight through the failures of a particular distribution.

Enter <a href="http://www.virtualbox.org/" target="_blank">VirtualBox</a>, an open source, company backed desktop virtualisation platform that runs incredibly smoothly on my desktop. XP installed without a hitch, and because the virtualisation is based on VT everything is extremely fast; the only code rewriting that happens is when an instruction doesn’t cause the right kind of trap and needs to be emulated. Networking runs beautifully (after telling XP about my local DNS server), and the sound card even works great with PulseAudio, which means I am slightly considering using iTunes again. (I like Amarok very much, but am annoyed that you can’t inspect playlists without adding them to the global playlist &#8211; if this feature makes it into 2.0 I’ll be very happy, but 2.0 has a long, long way to go). There’s a kernel module to install, but at rpm installation time this is done automatically, and if the kernel changes (as it did last night), VirtualBox detects this on the next start-up and recompiles the module for you.

The upshot of all this smoothness is that one side of the Compiz-enabled virtual desktop cube is an XP desktop, so things like Office and Visual Studio 2008 are one keystroke away. Having 2Gb of memory helps, as I gave 512Mb to XP and haven’t seen any slowdown. &#8211; I presume that the 512Mb is not permanently mapped anyhow.

I know that Xen is attacking an entirely different problem domain: operating systems research, strong isolation with as much sharing as possible, fault tolerance with migration and failover, all that good stuff. I work in the lab that originally produced it, and see daily that it’s an incredible piece of software. On the desktop, however, VirtualBox is a clear winner.