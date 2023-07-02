---
title: "You should prepare for the OpenSSL 3.x secvuln"
date: 2022-10-28
tags:
 - openssl
 - vuln
 - noxp
---

<xeblog-hero ai="Waifu Diffusion v1.3 (float16)" file="dark-sky-tokyo" prompt="cityscape, dark, red and black, monotone, black sky, smoke, tokyo"></xeblog-hero>

Hoooo boy, 2022 keeps delivering. It seems that the latest way things are
getting fun is that the [OpenSSL project announced a "CRITICAL" patch coming on
tuesday](https://mta.openssl.org/pipermail/openssl-announce/2022-October/000238.html)
for every release of OpenSSL that starts with `3.0`. The fixes will be released
as OpenSSL `3.0.7`. If you run OpenSSL `3.0.0` through `3.0.6`, you should
consider yourself vulnerable. I will cover how to check later in the post.

For people that only have casually followed the OpenSSL project, note that the
last time a "CRITICAL" patch was issued was to mitigate the
["Heartbleed"](https://en.wikipedia.org/wiki/Heartbleed) vulnerability. I am
going to split my analysis into two parts: facts and speculation.

## Facts

The patch to fix this issue will become public on Tuesday, November 1st. At the
time of writing, there is currently no publicly visible CVE identifier
associated with this issue. I am certain that one will become available on
November 1st.

The details on this issue seem to be super embargoed at the moment, but here is
what I can tell based on what has been done publicly:

- The [Tor project](https://www.torproject.org) has [urged relay owners to
  upgrade as soon as
  possible](https://forum.torproject.net/t/tor-relays-upcoming-openssl-3-security-bugfix-release/5330).
- The [release for Fedora 37 has been pushed back from October 25th to November
  15th](https://lwn.net/Articles/912776/).
- Popular tweeter [@SwiftOnSecurity](https://twitter.com/swiftonsecurity)
  [tweeted](https://twitter.com/swiftonsecurity/status/1585820615658938368)
  about hearsay about the severity of the issue and that tweet was deleted. No
  reason for the deletion was given.

<details>
  <summary>Contents of the deleted tweet</summary>

> The OpenSSL 3.x flaw will be significant, I have been told by someone in a
> position to know. Take your preparation seriously and prepare to act quickly –
> either patch or isolate.
> 
> It is sensitive enough they would not discuss specifics, out of respect for
> the embargo.
</details>

I am not a party to the embargo. I don't know how bad this is.

<xeblog-conv name="Cadey" mood="coffee">EDIT(2022 M10 30 16:28): It seems the Go
patches coming out on Tuesday are unrelated. I removed a section of the article
that mentioned an upcoming Go patch. It is an unrelated security patch
apparently.</xeblog-conv>

## Speculation

Based on the fact that two projects with TLS support in them are getting
security fixes on the same day, I'm thinking this is going to be _bad_. I am
anticipating having to spend the day patching systems and verifying that the
vulnerable version of OpenSSL (and the Go programming language) is not in use in
both production systems at work and on my personal infrastructure.

## Action items you can take

If you are an SRE, system administrator or otherwise going to respond to this
issue: now may be a good time to cancel plans for Halloween, or at least abstain
from drinking enough alcohol that you will become hung over or using substances
that will leave you exhausted the next day.

### Finding vulnerable programs

Now may be a good time to identify the systems in your control that use this
vulnerable version of OpenSSL. Here is one command you can use to find programs
using your distribution's package of OpenSSL:

```
sudo lsof -n | grep libssl.so.3
```

For example, here's what such a row could look like (extraneous details
snipped):

| command | pid | user | type | size/offset | node name |
| :------ | :-- | :--- | :--- | :---------- | :-------- |
| xesite  | 1740276 | cadey | mem | 6531231 | /nix/store/5nh3xmnx2lybwzl3p328q7b9rfh1ssyb-openssl-3.0.5/lib/libssl.so.3 |

From this you can tell that the code that powers my website loads openssl for
some reason, and that package will need to be updated.

On Ubuntu you can search for what packages own which files using `dpkg -S` like
this:

```
xe@nneka-sakuya:~$ sudo dpkg -S /usr/lib/x86_64-linux-gnu/libssl.so.3
libssl3:amd64: /usr/lib/x86_64-linux-gnu/libssl.so.3
```

You can get the version information of the package `libssl3` using `apt-cache
show` like this:

```
apt-cache show libssl3
```

This will show information [like
this](https://gist.github.com/Xe/b46dc36cb7b6db7e32389e9552ad40ba).

### Recording things

This is something you can do ahead of time and will help you triage this issue
before you apply fixes in production.

* Create a list of machines that you have administrative access to and run that
  command on each of them.
* Identify the version of OpenSSL that each program is using (or the version of
  OpenSSL the system is running) and use `systemctl 
  status` to trace things back to the corresponding services. Depending on facts
  and circumstances, this update may have effects on those services.
* Use this to create a list of potential "blast damage", or ways that services
  could fall over and cause issues. Communicate this information to support
  teams.

By now, you should have a list of all of the machines with vulnerable versions
of OpenSSL and all of the services that will need to be restarted.

<xeblog-conv name="Cadey" mood="coffee">Of course, if something _statically_
links OpenSSL for some reason, all bets are off. You may have to check the build
recipes of every bit of custom software you ship on your systems. Distribution
packages are usually good about using the system level dynamically linked
binaries, but most language-level package manager ecosystems grew around
distribution level packages so depending on the ecosystem in question you may
have to do a lot of digging.</xeblog-conv>

### Plan for patch tuesday actions

When the patches drop on Tuesday, you should do one of the two things:

* Update machines as _quickly_ as possible.
* Isolate machines from the Internet entirely until you can update the OpenSSL
  package on that system.

At this time it is not known if this is a _client_ or _server_ vulnerability. If
it is a _server_ vulnerability (one that affects services with OpenSSL being
used to encrypt data in-transit), then it is probably best to turn off
public-facing daemons, upgrade your packages, and then turn them back on. If
this is a _client_ vulnerability triggered by malicious servers then a lot of
things will change about the response process. You may have to copy the target
packages over SSH, install them manually, and then bring your external facing
services back up. Depending on how much you trust your network, it may be safe
to just install from your distributions package servers.

When your distribution tells you the version of the packages that have the
vulnerability fixed, record that as your target version of OpenSSL. Ensure every
system, docker image and virtual machine has OpenSSL updated. This may take a
while. You may want to split the work between teammates.

---

I am really hoping that this doesn't end up sucking, but I would probably plan
for it being about as bad as Heartbleed. Above all though, take care of yourself
and try not to stress out about this too much. Everyone on the Internet is going
to be as vulnerable as you are right now.

