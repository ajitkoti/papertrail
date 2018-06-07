---
id: 63
title: Adding a system call to the Linux Kernel
date: 2008-09-28T20:57:10+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=63
permalink: /adding-a-system-call-to-the-linux-kernel/
wp-syntax-cache-content:
  - |
    a:2:{i:1;s:647:"
    <div class="wp_syntax" style="position:relative;"><table><tr><td class="code"><pre class="c" style="font-family:monospace;">asmlinkage <span style="color: #993333;">long</span> sys_kstacksize<span style="color: #009900;">&#40;</span> <span style="color: #993333;">void</span> <span style="color: #009900;">&#41;</span>
    <span style="color: #009900;">&#123;</span>
    <span style="color: #b1b100;">return</span> THREAD_SIZE<span style="color: #339933;">;</span>
    <span style="color: #009900;">&#125;</span></pre></td></tr></table><p class="theCode" style="display:none;">asmlinkage long sys_kstacksize( void )
    {
    return THREAD_SIZE;
    }</p></div>
    ;i:2;s:1516:
    <div class="wp_syntax" style="position:relative;"><table><tr><td class="code"><pre class="c" style="font-family:monospace;"><span style="color: #339933;">#include &quot;../linux-2.6.25/include/linx/unistd.h&quot;</span>
    &nbsp;
    <span style="color: #993333;">int</span> main<span style="color: #009900;">&#40;</span> <span style="color: #009900;">&#41;</span>
    <span style="color: #009900;">&#123;</span>
    <span style="color: #993333;">long</span> stack_size<span style="color: #339933;">;</span>
    stack_size <span style="color: #339933;">=</span> syscall<span style="color: #009900;">&#40;</span> __NR_kstacksize <span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
    <span style="color: #000066;">printf</span><span style="color: #009900;">&#40;</span> <span style="color: #ff0000;">&quot;The kernel stack size is %ld<span style="color: #000099; font-weight: bold;">\n</span>&quot;</span><span style="color: #339933;">,</span> stack_size <span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
    &nbsp;
    <span style="color: #b1b100;">return</span> <span style="color: #0000dd;">0</span><span style="color: #339933;">;</span>
    <span style="color: #009900;">&#125;</span></pre></td></tr></table><p class="theCode" style="display:none;">#include &quot;../linux-2.6.25/include/linx/unistd.h&quot;
    
    int main( )
    {
    long stack_size;
    stack_size = syscall( __NR_kstacksize );
    printf( &quot;The kernel stack size is %ld\n&quot;, stack_size );
    
    return 0;
    }</p></div>
    ";}
categories:
  - Operating systems
tags:
  - hacking
  - kernel
  - linux
---
So, despite ostensibly being a &#8216;systems&#8217; guy, I haven&#8217;t spent too much time in my life getting hands on with the Linux kernel. I&#8217;ve written tiny toy operating-system-like projects before, but haven&#8217;t done much open-heart surgery on real life code.

I think this should change, so in my very limited spare time I&#8217;m doing some very simple projects to teach me more about the Linux kernel code layout so that if it should so happen in a job interview that someone asks me if I&#8217;m comfortable hacking at the kernel level I can answer \`yes&#8217; with far more conviction (I would probably answer positively anyhow, because I&#8217;m arrogant enough to think that it&#8217;s not beyond me, but there&#8217;s a lot of metaphorical difference between having the book on your shelf and having read it 🙂 ).
  
<!--more-->


  
The first important goal was to get a working development environment where I could boot kernels that I had built myself quickly and easily. After some searching for a minimal Linux system &#8211; but rejecting embedded systems, I wanted minimal in the sense that little else ran but the kernel, init and the shell &#8211; I decided upon [ArchLinux](http://www.archlinux.org/). Arch seems to be an absolutely no-nonsense, no bells and whistles kind of distribution, which is exactly what I was after. I didn&#8217;t want anything to obscure what was going on at the kernel level.

So I downloaded and installed Arch into a [VirtualBox](http://www.virtualbox.org/) virtual machine (as ever, VirtualBox works flawlessly), and got to work.

Building a kernel from source is very easy &#8211; download the source (I&#8217;m on 2.6.25), make config / make bzImage / make modules / make modules_install. Getting it to boot was more of a pain, simply because for some reason the kernel couldn&#8217;t mount the rootfs which is an ext3 filesystem. It turned out that at some point in the configuration step, the ATA IDE modules weren&#8217;t being selected even for building, and this obviously leads to problems when trying to mount a filesystem on a (virtual) IDE device. Running `mkinitcpio` gave the right hints when it said it couldn&#8217;t find the ATA module. Fixing that and rebuilding finally gave me a bootable kernel, and I was in a position to start hacking.

Knowing how labyrinthine things can get, I didn&#8217;t want to start with anything too taxing. So I went for the obvious OS 101 extension project: adding a system call. System calls are essentially the kernel&#8217;s API to userspace, and are called (on x86 at least) by loading the registers with the syscall number &#8211; Linux has 326 at the moment on x86 &#8211; putting the arguments on the stack and then invoking an interrupt which is trapped by the kernel. Once control passes to the kernel it can tell that a system call is to be called, find the right call point by indexing the system call number in a function table, and jumping to the actual code.

System calls are only added to the kernel when there is a compelling case to do so, as once added they can never be removed for fear of breaking compatibility &#8211; if you remove a syscall, you have to renumber all subsequently added syscalls and that breaks code. Therefore there&#8217;s not really much practical use in adding one, but a good deal of educational value!

The first thing I did was to write my syscall code. I didn&#8217;t really care what my syscall did, except I wanted its results to be verifiable so it needed to return some value. Going with an example from the book I am following ([Linux Kernel Development](http://www.amazon.co.uk/Linux-Kernel-Development-Novell-Press/dp/0672327201%3FSubscriptionId%3D08WX39XKK81ZEWHZ52R2%26tag%3Dws%26linkCode%3Dxm2%26camp%3D2025%26creative%3D165953%26creativeASIN%3D0672327201), highly, highly recommended) I settled for a simple call that returns the size of the kernel per-process stack, as defined by `THREAD_SIZE`. Here&#8217;s the code:

<pre lang="c">asmlinkage long sys_kstacksize( void )
{
    return THREAD_SIZE;
}</pre>

Very, very simple. `asmlinkage` just tells the compiler to find the arguments for the function (although there are none in this case) on the stack. Everything else is self explanatory. This goes in `kernel/sys.c` which is where a good deal of the other system calls are implemented.

Now the (relatively) tricky bit &#8211; plumbing the system call in such that when a call comes in for our syscall the kernel knows where to find it. This is architecture specific, but on x86 the only thing you have to do is to add it to `arch/x86/kernel/syscall_table_32.S`, right at the end of the list of extant syscalls. This list is assembled into a function call table such that the address of the syscall is at the offset of its syscall number. You can also infer the syscall number from this file as it&#8217;s well commented every five syscalls. My addition was number 327.

Then for completeness I added a `#define __NR_kstacksize 327` to `include/asm-x86/unistd_32.h` so that there was a corresponding lexical definition for the syscall number &#8211; you don&#8217;t really want to see syscalls called by number in code because you&#8217;ve got no idea what&#8217;s being called.

Quick re-build of the kernel to check that everything&#8217;s ok, then on to a user space test. Most syscalls are already plumbed in to userspace via libc or similar. However, a newly minted syscall won&#8217;t have any convenient userspace stubs, so we have to call it manually.

This is where my book let me down &#8211; or at least, understandably, hadn&#8217;t kept up with the last three years of kernel development since I bought it. There used to be macros that would generate userspace stubs for system calls called `_syscallN` for N arguments. They no longer exist. It took me a little while to find out what had replaced them, but it turns out that there is standard `syscall(2)` call that will invoke system calls on a userspace application&#8217;s behalf.

So, writing a tiny test program to invoke my new syscall was as easy as pie:

<pre lang="c">#include "../linux-2.6.25/include/linx/unistd.h"

int main( )
{
  long stack_size;
  stack_size = syscall( __NR_kstacksize );
  printf( "The kernel stack size is %ld\n", stack_size );

  return 0;
}
</pre>

Testing was even easier &#8211; I ran it once on the kernel that came with Arch and got -1 as a response, which is `ENOSYS` &#8211; an indication that syscall 327 didn&#8217;t exist. Rebooting into my newly rebuilt kernel tells me that the kernel stack size is 8192. Success!

Ok, so that&#8217;s a really small start. I&#8217;ve proved to myself at least that I can deterministically affect the behaviour of the kernel (it doesn&#8217;t always feel that way when you&#8217;re working with such a complex piece of software). My intended future projects are more ambitious: the next thing I&#8217;m going to do is to understand the scheduler, and perhaps replace it with a massively less efficient one. Then I&#8217;m off into subsystem land &#8211; I want to write a small filesystem (an in memory one might be a simple start) and then figure out the IP stack. Should be enough to keep me busy on Sunday evenings for a while!