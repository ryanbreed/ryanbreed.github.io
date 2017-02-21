---
layout: post
date: 2017-02-21
published: true
title: "eBPF Ambush - Strange Ways"
description: "developing a dynamic surveillance capability with eBPF"
category: post-entry
tags:
  - eBPF
  - paystub
  - stix:TTP-e1e4e3f7-be3b-4b39-b80a-a593cfd99a4f
  - DOOM, MF

excerpt_separator: <!--more-->

---
{% include JB/setup %}

We're going to expand on the [previous example] of using endpoint 
instrumentation to develop a semantic sensor that provides low noise and a 
highly dynamic diagnostic capability. Unfortunately, it requires a visit to 
the bleeding edge of kernel and compiler infrastructure to fully exploit, 
but the tooling is starting to catch up to the mainstream, so it's worth 
learning about now.

<!--more-->

### Theory
<blockquote>
<p>Wreak havoc, *beep beep* it's mad traffic</p>
<a href="http://www.youtube.com/watch?feature=player_embedded&v=uSxlZQUqVPY">
<img src="http://img.youtube.com/vi/uSxlZQUqVPY/0.jpg" alt="strange ways" width="240" height="180" border="10" />
</a>
</blockquote>

Linux Kernel versions since 3.15 have a new tracing facility called
[eBPF], which allows you to attach tiny bits of code to various events
in kernel and user space. This creates an amazingly powerful capability 
to inspect (and modify) pretty much anything as processes interact with 
the kernel or userland code. Think hooking but without messing with the
actual call chain from the perspective of the executing process. The 
rabbit hole is deep on this one.

Building these instruments requires programming in assembly for a virtual 
machine executing in the kernel. The basic idea is an extension of the
original BPF socket filtering done for things like tcpdump, but extended
to all function invocations (and some non-function event triggers). The 
[kernel docs] for the raw interface are a bit inscrutable, but luckily 
the toolchains are being built to make things easier. One such project is 
[bcc], which has a bunch of sample code for building various tracing 
instruments in c, python, and lua. 


#### The Setup

We've still got our example app [paystub]. This time, we're going to leave
the application install alone and even use the OS scripting runtime to 
host the app. We can do this because eBPF allows us to trace everything
from inside the kernel and be highly selective about what calls we attach 
to.

You'll need to start out with a linux distro that has one of the more
modern kernel. This [support matrix] will give you an idea of what kernel
versions are available for experimentation. I used a fedora25 instance
for building the toolchain and hosting the paystub app. The [install docs]
are pretty comprehensive - I compiled from source on fedora because the
binary installs were a little hinky, but I suspect it's much more 
straightforward on ubuntu. 

Since we're just fooling around, we'll just use the tools directly from
the installation and save the opportunity to develop more general-purpose
tooling for a later date. For this example, we'll primarily be looking at
[execsnoop].

#### The Experiment

With paystub already running, we're going to trace kernel execs and
see what happens with another meterpreter injection. Firing up the 
tool is as simple as running them from a root process:

    cd /usr/share/bcc/tools && ./execsnoop

This will start a system-wide trace of all exec(2) invocations, printing
the pid, timestamp, and arguments. You might want to do this on a 
quiet instance because things can get busy with cron jobs and the like.
The cli args do let you filter things down by pid or process name if
you know what you're looking for ahead of time.

#### The Payoff

We're going to run the same intrusion from last time - fork a bindshell
and then upgrade to a meterpreter session. 

<pre class="highlight"><code>
sudo ./execsnoop
OMM              PID    PPID   RET ARGS
sh               7928   7919     0 /bin/sh -c echo 1725256335;echo AuyqFdviELSMMpNxXoZdDZPNiNnvMkUm
sh               7930   7919     0 /bin/sh -c uname -ms;echo atOMXzDhJPzOTtnVSkNESnydVjXRsGba
uname            7931   7930     0 /usr/bin/uname -ms
sh               7933   7919     0 /bin/sh -c echo -n f0VMRgEBAQAAAAAAAAAAAAIAAwABAAAAVIAECDQAAAAAAAAAAAAAADQAIAABAAAAAAAAAAEAAAAAAAAAAIAECACABAibAAAA4gAAAAcAAAAAEAAAMdv341ND
which            7936   7935     0 /usr/bin/which base64
base64           7937   7935     0 /usr/bin/base64 --decode -
chmod            7938   7933     0 /usr/bin/chmod +x /tmp/FPaQc
FPaQc            7939   7933     0 /tmp/FPaQc
sleep            7940   7933     0 /usr/bin/sleep 2
rm               7941   7933     0 /usr/bin/rm -f /tmp/FPaQc
rm               7942   7933     0 /usr/bin/rm -f /tmp/cFfFG.b64
</code></pre>

Boom. Pretty neat. We can see the base64 meterpreter being written out to /tmp,
being execed, and then removed after the image has been launched and is
running. 

#### Discussion and Other Uses

Obviously, this is a neat trick to have up your sleeve if you happen to be
deploying production apps onto a modern kernel-based platform. At this point,
its utility for incident response is very promising, as it allows you to 
sidestep some of the messier aspects of ptrace or rooting around in memory.

The docs for the rest of the tools point to some pretty interesting
avenues of exploration. In particular:

* [sslsniff] traces reads and writes from OpenSSL and gnutls
* [trace] is a handy wrapper for probing any syscall, its arguments, and return values
* [bashreadline] will print out commands as they are entered into any shell
* [ttysnoop] will sniff output written to pty char devices

*WARNING: OPINION* - this is where I'd normally get back on my soapbox about
leaning on automated testing and deployment as a critical security enabler 
for rolling out capabilities like this in production. If you're not there, 
this isn't necessarily something I'd say merits slapping something together 
just to get to kernel 4.9+ deployments. The tooling is pretty immature, and 
the potential adversary countermeasures haven't really been explored yet, so 
It's not a capability I'd break the [error budget] for just to get rolled out. 

Yet.

More reading here: [dive into bpf](https://qmonnet.github.io/whirl-offload/2016/09/01/dive-into-bpf/)


<blockquote>
Who told you that? Rolled through, BRRRAT
<a href="http://www.youtube.com/watch?feature=player_embedded&v=qEjz7gl0D7E">
<img src="http://img.youtube.com/vi/qEjz7gl0D7E/0.jpg" alt="strange ways" width="240" height="180" border="10" />
</a>
</blockquote>


[previous example]: https://ryanbreed.github.io/post-entry/2017/02/trap-goin-crazy
[eBPF]: https://events.linuxfoundation.org/sites/events/files/slides/bpf_collabsummit_2015feb20.pdf
[kernel docs]: https://www.kernel.org/doc/Documentation/networking/filter.txt
[bcc]: https://github.com/iovisor/bcc
[paystub]: https://github.com/ryanbreed/paystub-sinatra
[support matrix]: https://github.com/iovisor/bcc/blob/master/docs/kernel-versions.md
[install docs]: https://github.com/iovisor/bcc/blob/master/INSTALL.md
[execsnoop]: https://github.com/iovisor/bcc/blob/master/tools/execsnoop_example.txt
[sslsniff]: https://github.com/iovisor/bcc/blob/master/tools/sslsniff_example.txt
[trace]: https://github.com/iovisor/bcc/blob/master/tools/trace_example.txt
[bashreadline]: https://github.com/iovisor/bcc/blob/master/tools/bashreadline_example.txt
[ttysnoop]: https://github.com/iovisor/bcc/blob/master/tools/ttysnoop_example.txt
[error budget]: https://www.usenix.org/conference/srecon15/program/presentation/alvidrez
