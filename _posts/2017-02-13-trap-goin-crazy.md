---
layout: post
date: 2017-02-14
published: true
title: "Trap Goin Crazy"
description: "developing a tactical surveillance capability with auditd"
category: post-entry
tags:
  - auditd
  - paystub
  - stix:TTP-e1e4e3f7-be3b-4b39-b80a-a593cfd99a4f
  - Mane, Gucci
  - 40, E-

excerpt_separator: <!--more-->

---
{% include JB/setup %}

Watching the wires is an endeavor fraught with peril and uncertainty.
Detecting exploit activity with only network instrumentation has a
marginal benefit that diminishes rapidly once your threat profile
begins to include mildly sophisticated actors. Here, we take a look at
some relatively limited and simple configuration profiles that can
be applied to a production server that make post-exploitation activity
loud and obvious.

<!--more-->

### Theory
<blockquote>
<p>
  <a href="https://genius.com/1614280/Snoop-dogg-candy-drippin-like-water/Whatchu-tryna-buy-pimpin-yknow-its-kinda-dry-here">
    Whatch&#39;all try&#39;na buy, pimpin? y&#39;know it&#39;s kinda dry here
  </a>

  <a href="http://www.youtube.com/watch?feature=player_embedded&v=mKli0y-Xr-Q&t=28s">
    <img
      src="http://img.youtube.com/vi/mKli0y-Xr-Q/0.jpg"
      alt="CANDY" width="240" height="180" border="10"
    />
  </a>
</p>
</blockquote>
Building control surfaces to stop entry operations 'left of bang' is certainly
where most defenders mentally adjust their mindset so they can sleep at night.
As discussed [previously], this strategy is at best fruitless in the face of
an advanced actor, and at worst the actual cause of vulnerability in
otherwise defensible infrastructure.

However, once entry has been achieved, the aggressor must use the target's
infrastructure to conduct subsequent maneuver or achieve the intended objective.
This parasitic use of the defender's infrastructure is itself a critical
vulnerability and the one which we shall exploit to create some high quality
instrumentation and alerting.

#### The Setup

We're a defender tasked with protecting a horrible application called [paystub].
We've been drinking the devops koolaid, so we've got a [CD] pipeline to
push deployments and manage configuration of our application. The first thing
we're going to do is [extract the application runtime] so that we have a ruby
interpreter that is dedicated for running our app and our app alone.

Now our (userspace) surface exposure has been separated from the OS, and
nothing else on the box has any reason to touch any part of the application
runtime. Once this configuration can pass integration testing and is deployable,
we can begin its instrumentation.

#### Targeted auditd Policy in Managed Deployments
> oh yeah, that thing

[auditd] is the userspace component of the kernel audit subsystem. It is
responsible for setting the policy for auditable events, receiving them
from kernelspace, and writing them to the appropriate file/network channels.
It's got plenty of tricks up its sleeves, but the one we're going to use
is its capacity to audit system calls.

The basic system call audit rule looks something like this:

    -a always,exit -F arch=b64 -S listen  -F filterkey=filterval -F key=some-searchable-tag

This audit rule sets a trigger to always fire off a log entry when the
`listen` call returns an exit value. You can specify an arbitrary number of
filter conditions based on things like uid, selinux context type, etc. Since
we've already separated our application runtime out to a known set of binaries,
we can use that as a filter condition to set triggers only on our exposed app
and watch for things that should never occur. For example:

* Opening up another listening socket on a different port
* Initiating a network connection
* Exec(3)ing another binary
* Forking off another process
* Being denied access to read or write a file
* and so on

Here's a sample 64bit policy just to whet your appetite:

    -a always,exit -F arch=b64 -S accept -F exe=/path/to/ruby -F key=64bit-ruby-ioc
    -a always,exit -F arch=b64 -S connect -F exe=/path/to/ruby -F key=64bit-ruby-ioc -F exit!=-EINPROGRESS -F exit!=-ENOENT
    -a always,exit -F arch=b64 -S listen -F exe=/path/to/ruby -F key=64bit-ruby-ioc
    -a always,exit -F arch=b64 -S execve -F exe=/path/to/ruby -F key=64bit-ruby-ioc
    -a always,exit -F arch=b64 -S fork  -F exe=/path/to/ruby -F key=64bit-ruby-ioc
    -a always,exit -F arch=b64 -S vfork -F exe=/path/to/ruby -F key=64bit-ruby-ioc
    -a always,exit -F arch=b64 -S bind  -F exe=/path/to/ruby -F key=64bit-ruby-ioc
    -a always,exit -F arch=b64 -S dup2  -F exe=/path/to/ruby -F key=64bit-ruby-ioc
    -a always,exit -F arch=b64 -S pipe2 -F exe=/path/to/ruby -F key=64bit-ruby-ioc
    ## ignore unix domain connections
    -a never,exit -F arch=b64 -S socket -F a0=1
    ## ignore netlink connections
    -a never,exit -F arch=b64 -S socket -F a0=16
    ## This should be everything else
    -a always,exit -F arch=b64 -S socket -F exe=/path/to/ruby -F key=64bit-ruby-ioc

#### The Payoff

Here's some sample logs of what a dialed-in audit configuration can produce
from a box that's getting a reverse meterpreter inserted:

<pre class="highlight"><code>
2017-01-19T01:17:14-06:00 type=SYSCALL msg=audit(1484810234.635:2088): arch=c000003e syscall=58 success=yes exit=1309 a0=7f40ed08d2f4 a1=511 a2=fffffaef a3=8 items=0 ppid=1082 pid=1297 comm="server.rb:285" exe="/app/ruby/runtime/bin/ruby"  key="stub-ruby-debug" ARCH=x86_64 SYSCALL=vfork UID="app-ruby" GID="app-ruby"

2017-01-19T01:17:14-06:00 type=SYSCALL msg=audit(1484810234.643:2089): arch=c000003e syscall=268 success=yes exit=0 a0=ffffffffffffff9c a1=1adb0f0 a2=1fd a3=7ffe6d3ccb50 items=1 ppid=1309 pid=1314 auid=1001 uid=1001 gid=1001 comm="chmod" exe="/usr/bin/chmod"  key="perm_mod" ARCH=x86_64 SYSCALL=fchmodat  UID="app-ruby" GID="app-ruby"

2017-01-19T01:17:14-06:00 type=PATH msg=audit(1484810234.643:2089): item=0 name="/tmp/ZikMi" inode=737857 dev=fd:00 mode=0100664 ouid=1001 ogid=1001 rdev=00:00 objtype=NORMAL OUID="app-ruby" OGID="app-ruby"

2017-01-19T01:17:14-06:00 type=SYSCALL msg=audit(1484810234.644:2090): arch=40000003 syscall=102 per=400000 success=yes exit=3 a0=1 a1=ffc45234 a2=0 a3=0 items=0 ppid=1309 pid=1315 auid=1001 comm="ZikMi" exe="/tmp/ZikMi"  key="ioc-32bit-abi" ARCH=i386 SYSCALL=socketcall UID="app-ruby" GID="app-ruby"

2017-01-19T01:17:14-06:00 type=SYSCALL msg=audit(1484810234.644:2091): arch=40000003 syscall=102 per=400000 success=yes exit=0 a0=3 a1=ffc45224 a2=0 a3=0 items=0 ppid=1309 pid=1315 auid=1001 comm="ZikMi" exe="/tmp/ZikMi"  key="ioc-32bit-abi" ARCH=i386 SYSCALL=socketcall UID="app-ruby" GID="app-ruby"

2017-01-19T01:17:14-06:00 type=SOCKADDR msg=audit(1484810234.644:2091): saddr=02001151AC1401EF010000000000000001000000A265C4FF00000000AD65C4FF2A66C4FF3B66C4FF5166C4FF6966C4FFA366C4FFB366C4FFBE66C4FFCC66C4FFED66C4FF0867C4FF1B67C4FF2967C4FFC56CC4FFF26CC4FF076DC4FFAC6DC4FFCA6DC4FFE86D SADDR={ fam=inet laddr=172.20.1.239 lport=4433 }
</code></pre>

[This ansible playbook] is a starting point for getting your feet wet. It's got
a very limited set of [targeted rules] that won't fire for processing regular
http requests, but do fire once you start using any of the payload invocation
functions. It's not quite complete, but it's a good start for figuring out
a production configuration that works for you. Remember that for each system
call you audit, there could be both 32 and 64bit versions, and that either one
could be in use.  

#### Caveats
auditd has lots of quirks. Not the least of which is that some of the default
policies can produce oceans of useless data. Two of the more insidious quirks
are the [disk logger] and [event dispatcher] overflow handling settings. If
either process gets too backed up, it could `suspend`, and the process will
still be running, but it will no longer process events. Hilarious. Other
settings could cause silent drops, or shouts into the void if you're pushing
audits into a syslog infrastructure that's also having troubles.

I'd recommend using a canary process to regularly pulse logs through the audit
subsystem to ensure its ongoing functionality if this is a monitoring control
you're seriously considering.

[paystub]: https://github.com/ryanbreed/paystub-sinatra
[previously]: /doctrine/2017/02/mapping-out-adversary-coas
[auditd]: https://github.com/linux-audit/audit-userspace
[CD]: https://martinfowler.com/bliki/ContinuousDelivery.html
[extract the application runtime]: https://github.com/ryanbreed/packages/blob/master/defs/ruby233.sh
[this ansible playbook]: https://github.com/ryanbreed/paystub-playbooks/blob/master/paystub-sinatra.yml
[targeted rules]: https://github.com/ryanbreed/paystub-playbooks/blob/master/templates/50-runtime-ruby.rules.j2
[disk logger]: https://linux.die.net/man/8/auditd.conf
[event dispatcher]: https://linux.die.net/man/5/audispd.conf
