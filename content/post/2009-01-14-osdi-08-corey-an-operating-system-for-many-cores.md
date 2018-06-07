---
id: 133
title: "OSDI '08: Corey, an operating system for many cores"
date: 2009-01-14T22:50:19+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=133
permalink: /osdi-08-corey-an-operating-system-for-many-cores/
categories:
  - computer science
  - Operating systems
  - Paper Walkthrough
tags:
  - computer science
  - multicore
  - Operating systems
  - osdi
  - paper review
---
Just before Christmas, the systems community held one of its premier conferences - Operating Systems Design and Implementation (OSDI '08). This biannual conference showcases some of the best research in operating systems, networks, distributed systems and software technology from the past couple of years.

Although I wasn't lucky enough to go, I did grab a copy of the proceedings and had a read through a bunch of the papers that interested me. I plan to post summaries of a few to this blog. I see people ask repeatedly on various forums (fora?) "what's new in computer science?". No-one seems to give a satisfactory answer, for a number of reasons. Hopefully I can redress some of the balance here, at least in the systems world.

Without further ado, I'll get stuck in to one of the OSDI papers: [Corey: an operating system for many cores](http://www.mit.edu/~y_z/papers/corey-osdi08.pdf) by Boyd-Wickizer et al from a combination of MIT, Fudan University, MSR Asia and Xi'an Jiaotong University (12 authors!). Download the paper and play along at home, as usual.

<!--more-->



Almost every systems presentation you see nowadays starts off with the same observation: that CPU clock rates are stagnating and the way to improve performance is now to scale horizontally by adding more cores to the machine, rather than vertically by increasing the speed of the CPU. The 'free lunch' that CPU designers gave application programmers has finally ended, and now we have to figure out how to write programs for multiple processors.

This is not an easy problem to solve. We now have to structure our programs to run in parallel, rather than sequentially. The problem with running code in parallel is that it is hard to efficiently control access to shared resources between multiple threads. Concurrent access to, for example, main memory is fraught with all kinds of difficulties. Unless the order of access is very strictly controlled, processes may mutually wait on a resource that another holds, causing deadlock, or they may observe an inconsistent view of a shared variable, or they may corrupt a shared data structure, to mention only three.

Mechanisms to mediate access to shared resources have been developed since at least the late 1960s by computer scientists. Dijkstra introduced the idea of semaphore variables which act as locks that allow access to a resource by a controlled number of concurrent threads. However, locks are problematic. Getting a locking protocol correct is hard to do efficiently - and tremendously hard to debug. Naieve locking protocols are easier (but certainly not easy) - and yet they typically harm performance by being over-conservative. What happens in this case is that one process waits patiently to get access, when it could be doing productive work. Ideally we want to minimise the amount of time that a resource is held for, but this complicates the locking protocol again.

These problems are especially acute when writing an operating system. Standard operating systems have centralised data structures representing, for example, the page table for virtual memory or a list of runnable processes. If the operating system is to run on a multi-core machine then the operating system code must mediate access to these shared structures to ensure their consistency. Performance is, clearly, important as much time is spent inside operating system code.

The Corey operating system takes an inverted approach: instead of sharing structures between cores by default, every system structure should be private to a core unless an application tells the OS to share it. This private-by-default approach goes some way towards minimising the amount of sharing that an OS has to do, and therefore improves the concurrency of the OS at the cost of foisting some complexity onto the application programmer.

## Exokernels

Corey is built along the lines of the _Exokernel_ project, which is an operating design proposed by Dawson Engler and others. There's not enough space here to go into a lot of detail about the Exokernel, but the interested are pointed towards [this paper](http://pdos.csail.mit.edu/papers/exokernel-sosp95.ps), and [this SOSP '97 paper](http://pages.cs.wisc.edu/~remzi/Classes/736/Fall2007/Papers/exo-sosp97.pdf) which covers some lessons learned about exokernel design.

The basic idea of the exokernel approach is that the operating system should be responsible only for multiplexing secure access to the hardware. Any other abstractions, such as virtual memory, schedulers or filesystems, are left to "library operating systems" to implement in 'user space', although that's a less clear distinction than with more traditional kernel designs. Exokernels check only if a resource is free to use and if an application is allowed to access it.

The rationale is that performance and flexibility is limited by forcing applications to use a uniform set of abstractions. The idea is that the operating system only has to mediate access to resources, and that other abstractions can be layered on top without conflicting with each other.

Note that this implies that there is \*no\* centralised control in exokernels. No entity has global knowledge about the system (save for that exposed by certain kernel objects), and therefore no entity makes decisions based on a complete view of the current system state. Instead, applications - linked to library operating systems - make local decisions. This implies that exokernels may be very well suited for running dedicated sets of tasks - such as several servers - more than the arbitrary workloads of desktop operating systems. However, this is certainly an arguable point.

The API for dealing with an exokernel is therefore radically different to the standard userspace API for a traditional kernel. The XoK project, and Corey similarly, expose a number of kernel data structures that allow for access to machine features. The innovation of Corey is that these structures do not have to be shared between processes via the kernel, and therefore easy concurrency is possible, and contention fo shared structures should be dramatically reduced.

## Kernel Structures

Corey runs a kernel on each core in a multicore machine. Each kernel collaborates to keep some shared information accurate, but as much as possible is kept kernel-local.

Corey kernels maintain information about every kernel data structure, called 'objects'. This metadata is shared between kernels as it is permanently in mapped physical memory. Shared access is mediated traditionally, via spin-locks and read-write locks. Therefore every kernel can see a list of ojects in the system, and some limited metadata about the object, even if they cannot explicitly manipulate the object itself.

Five types of objects are exposed by a Corey kernel: shares, segments, address spaces, cores and devices.

### Shares

Shares are used by Corey kernels to lookup the physical address of object metadata by that object's id. As the paper says, a global table of object->address mappings would suffer unnecessarily from contention, so each kernel keeps a private set of lookup tables, called shares. A kernel may only manipulate an object that is present in that kernel's shares.

As their name suggests, shares may be shared between kernels. A library operating system may arrange for each kernel to have a private share and one global share; or they may organise for a more complex sharing scheme. When an object is allocated by a kernel, the kernel is told which share (of the ones it may access) to place the object in. This gives a hint to the kernel about the potential level of contention for that object, which may - this is speculation - allow for specialised code to be used for accessing the object which is optimised for the particular concurrent case.

### Segments

Segments are the low-level abstraction to physical memory. A segment may be shared between kernels, or accessed only by a single kernel. Since different cores will have different caches (in general), it's important for the kernel to know how a segment is shared so that the correct cache management protocol can be run. Segments may be marked as copy-on-write or copy-on-reference so that they can make efficient usage of physical memory.

### Address Spaces

Address spaces are represented in Corey by _address trees_ that allow for fine-grained sharing of address spaces between cores. Each kernel maintains a root address tree, which maps addresses either to segment objects or to another address tree, which may be shared between cores. In that case the same address maps to the same segment (or, I suppose, a non-mapped page). The argument made in the paper is that coarse sharing (i.e. shared everything or shared nothing) don't meet the demands of most applications and impedes scalability.

### Cores

Remember that the kernel only concerns itself with mediating access to shared resources, and doesn't have anything to say about which processes run on which core. Therefore the core object is provided to allow library operating systems to initiate execution themselves. Cores are instructed to run code given a context consisting of a set of shares, pointer to the instruction to start from, a pointer to a stack segment and an address tree. The execution then continues indefinitely (barring crashes) until either the core itself or some other core instructs it to halt (which means being able to access to core in the available kernel shares).

One aspect in which this offers some new flexibility is scheduling. There is no kernel state associated with scheduling different tasks to run. Instead, the applications themselves can implement a custom scheduler - but only if required to do so; they can instead keep a core all for themselves. Again, this fine-grained control of system resources allows great flexibility without sacrificing performance.

### Devices

Devices are exactly that, kernel objects representing in-machine devices such as peripherals, network cards and so on. Communication between devices and applications occurs through shared segments. Each segment has a header that contains enough information to communicate requests to the device - for example, a network device segment header would contain a transmit bit that would be set by an application when it wantedto send a packet.

Quite how the layout of this header is standardised is not clear - whether it's device specific (and therefore requires each library operating system to have its own device specific drivers) or whether the kernel itself takes care of the smallest amount of abstraction.

## From Kernel Object to Useful Abstractions

There's clearly a lot left unsaid by the set of objects that Corey offers to applications. Pretty much all of the standard abstractions that applications have traditionally come to rely upon aren't available: there's no concept of files, sockets, virtual memory, scheduling, multiplexed access to devices... what you get is a very, very bare set of bones. However, this approach gives exokernels, and Corey in particular their power: computers can run completely different operating systems on different cores without having to worry too much about their interactions. If your webserver needs one set of abstractions but your packet filter needs another, exokernels can support both without having to go the full distance towards virtualisation. If all you are doing is copying packet headers into memory and filtering them based on their source address, do you really need a file system or a scheduler?

Corey is sufficiently flexible that if you do need these abstractions, they can be built. The paper describes briefly how time-multiplexed scheduling might be implemented. In more detail, the authors describe the standard library operating system that they have used to build test systems and run their experiments. This provides an augmented <tt>fork</tt> system call, a shared buffer cache that's implemented with a lock-free tree (which makes disk access quicker and easier to make correct) and a network stack which is mutiplexed down onto the underlying hardware. Many different network stacks may be run in parallel, since the code is not in the core kernel.

## Evaluation and Conclusions

Corey is tested on two large systems - the [

Phoenix](http://csl.stanford.edu/~christos/sw/phoenix/) multicore-MapReduce implementation and a bespoke web server application. MapReduce allocates a lot of memory at the map stage to store intermediate results, and this therefore stress-tests the virtual memory subsystem. Corey outperformed a Linux version by up to 10% for multiple cores (although suffered by comparison for less than 4 cores). The paper contains a great discussion of exactly why this should be, but the main message is that the Linux version spent over 10% of its time flushing out the Translation Lookaside Buffer (TLB) when a virtual address was unmapped by a worker thread. Since Corey does not explicitly share virtual addresses by default, there is no need to wipe the TLB for unshared segments since Corey knows that the mapping is not present in any other core's TLB except for the one that unmapped the segment. The reclamation is caused by libpthread unmapping thread-local stack pages - which clearly have no need to be shared. Invalidating remote TLBs is an extremely expensive operation.

The evaluation also shows that selective sharing is important to achieving scalability. When sharing is turned off completely, performance scales badly as every time a page is accessed that belongs to another core, that core has to be accessed to get at the page mapping. This is again very expensive.

The summary message then is that fine-grained control of address-space sharing is important for scalability. Sharing an address space is not free - synchronising the mappings can be expensive - but that can be outweighed by the benefits accrued from avoiding repeated cross-core communication.

The second benchmark demonstrates the benefits of assigning workloads to dedicated cores. Of particular interest is the result that shows Corey scaling nearly perfectly for a web server benchmark that simply accepts connections, writes 128 bytes and closes the connection. Linux sees a dropoff in throughput scalability after about 4 cores - this is clearly a big win for the Corey design.

The authors emphasise that Corey is a prototype design. The thing I like most about Corey is, paradoxically, also my biggest criticism of the paper. The exokernel design is great, but the cost of rewriting applications to target it is not really discussed. It's not also that easy to tell how much benefit has come from the move to an exokernel - and therefore dispensing with unnecessary abstractions - and how much has come from the share-nothing-by-default approach that Corey has taken. The delta that Corey gives over XoK is not apparently huge, although the amount of work that has gone into this paper is staggering. It would have been nice to see how Corey's kernel objects differ from XoK's - I'm only familiar in passing with the original exokernel work.

The improvements that MapReduce got, although significant, weren't enormous. I imagine that they'd be predicted to increase as the number of cores increases further, but I would like to have seen some analysis of whether the point at which the gains flatten out has just been pushed further away (which is still a win) or whether the scaling shown will continue indefinitely. Is there any more concurrency that can be squeezed out of these applications? The choice of kernel objects, while clearly principled, isn't exactly explained. Might there be other abstractions that help out applications even more? If we're going to the trouble of rewriting everything, we can experiment wildly with the APIs.

It's really good to see people building operating systems, as the status quo has reigned for a while and most results seem to be implemented on either Linux or Windows. This was definitely one of the most enjoyable papers from OSDI '08.
