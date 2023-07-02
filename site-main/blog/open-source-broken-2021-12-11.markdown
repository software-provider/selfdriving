---
title: '"Open Source" is Broken'
date: 2021-12-11
author: Mai
tags:
 - rant
---

or: Why I Don't Write Useful Software Unless You Pay Me

Recently there was a [massive
vulnerability](https://www.lunasec.io/docs/blog/log4j-zero-day/) found in a
critical Java ecosystem package. When fully weaponized, this allows attackers to
coerce Java servers into executing arbitrary code that was fetched from an LDAP
server.

[If this is news to you and you work at a Java shop, I'm sorry but you have a
long couple days ahead.](conversation://Mara/hacker)

I believe this is a perfect microcosm of all of the major ecosystem problems
with "Open Source" software. I have some thoughts about all this, as I think
log4j2 is a _perfect_ example of one of the worst case scenarios for this. It is
perfectly reasonable for everyone involved in this issue to have done all this
for perfectly valid solutions to real-world problems and this also to have
created a massive hole on accident in the process.

<center>

![the XKCD comic "Dependency", depicting all modern digital infrastructure being
held up by some random project made by a thankless anonymous person in
Nebraska.](https://imgs.xkcd.com/comics/dependency.png)

[XKCD #2347: Dependency](https://xkcd.com/2347/)

</center>

All software is made on top of the shoulders of giants. Consider something as
basic as running an SSH server on the Linux kernel. In the mix you would have at
least 10 vendors (assuming a minimal Alpine Linux system in its default
configuration), which means that there are at least 10 separate organizations
that still have bills to pay with actual money dollars regardless of the number
of users of the software they are giving away for free. Alpine Linux is also a
great example of this because it is used frequently in Docker contexts to power
many, many companies in production. How many of those companies do you think
fund the Alpine Linux project? How many of those companies do you think even
would even THINK about funding the Alpine Linux project?

I've had this kind of conversation with people before and I've gotten a
surprising amount of resistance to the prospect of actually making sure that the
random smattering of volunteers that LITERALLY MAKE THEIR COMPANY RUN are able
to make rent. There is this culture of taking from open source without giving
anything back. It is like the problems of the people who make the dependencies
are irrelevant.

<center>

![A meme based on the Tim and Eric "It's free real estate" template contrasting
the idea of open source software maintained by passionate developers with a
heartless taking without giving attitude](/static/blog/5xi3x7.jpg)

</center>

GitHub stars famously cannot be used to pay rent. An example of this is the
[`core-js` debacle](https://github.com/zloirock/core-js/issues/767). `core-js`
is a JavaScript library that gives JavaScript's standard library a lot of core
primitives that can make you not need to reach out to other libraries. This
library is also infamous for letting you know that the author is looking for a
job every time you install it in CI. You probably have seen this message in your
CI a thousand times:

```
Thank you for using core-js ( https://github.com/zloirock/core-js ) for
polyfilling JavaScript standard library!

The project needs your help! Please consider supporting of core-js on Open
Collective or Patreon:
> https://opencollective.com/core-js 
> https://www.patreon.com/zloirock 

Also, the author of core-js ( https://github.com/zloirock ) is looking for a
good job :-)
```

The author of the project is either still in prison for vehicular manslaughter
or has just been released. `core-js` is a dependency of React. How many of you
have actually donated to this project? Especially if you use React?

Now let's turn our eyes to `log4j2`. This project is effectively in the standard
library for Java users. This library is so ingrained into modern Java that
you'd expect the developers of it would be well-funded and not need to focus on
anything else but that library, right?

No.

<center><blockquote class="twitter-tweet"><p lang="en" dir="ltr">This is the maintainer who fixed the vulnerability that&#39;s causing millions(++?) of dollars of damage.<br><br>&quot;I work on Log4j in my spare time&quot;<br>&quot;always dreamed of working on open source full time&quot;<br>&quot;3 sponsors are funding <a href="https://twitter.com/rgoers?ref_src=twsrc%5Etfw">@rgoers</a>&#39;s work: Michael, Glenn, Matt&quot;<br><br>People, what are we doing. <a href="https://t.co/2hAxUWCjuC">pic.twitter.com/2hAxUWCjuC</a></p>&mdash; Filippo ${jndi:ldap://filippo.io/x} Valsorda (@FiloSottile) <a href="https://twitter.com/FiloSottile/status/1469441487175880711?ref_src=twsrc%5Etfw">December 10, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></center>

As of yesterday, there were a grand total of three sponsors for this person's
work. THREE. As of today, this number is now 14; however this is no excuse. This
person should be funded in a level that is appropriate for how critical `log4j2`
is used in the ecosystem. There is no excuse for this. This person's _spare time
passion project_ is responsible for half of the internet working the way it
should. Vulnerable companies to this issue included Apple, Google, my cell phone
carrier and basically everyone that uses JavaEE in its default configuration. 

[Seriously, I could trigger some part of my cell carrier's infra reaching
out to a DNS server with a specially crafted SMS
message.](conversation://Cadey/facepalm)

If `log4j2` is responsible for your company's success, you have a moral
obligation to [donate to the person who creates this library
thanklessly](https://github.com/sponsors/rgoers).

[As for the problem that created this vulnerability in the first place: what
where they THINKING when they allowed user-submitted untrusted strings to
contain JDNI references that would then cause the JVM to load arbitrary bytecode
into ram and then run it without having to specify that in the format string to
begin with? Like why would you even need to do that in the _user-supplied_ part
of the format string? What would this even accomplish other than being a great
way to get a shell whenever you wanted?](conversation://Numa/stare)

There is a friend of mine who has been thanklessly maintaining an online radio
station stack for a long time. He has been abused by his users. Users will throw
5 bucks in the tip jar and then get very angry when he doesn't drop everything
and fix their incredibly specific problems on a moment's notice. He has tried to
get jobs at places, but every time they keep trying to screw him out of
ownership of his own projects and he has to turn them down. Meanwhile the cash
bleed continues.

This is why I am very careful about how I make "useful" software and release it
to the world without any solid way for me to get paid for my efforts. I simply
do not want to be in a situation where my software that I develop as a passion
project on the side is holding people's companies together. That's why I make
software how and where I do. Like, no offense, but I really do not want to go
unpaid for my efforts. The existing leech culture of "Open Source" being a pool
of free labor makes it hard for me to want to have my side projects be actually
useful like that unless you pay me.

[Okay, part of this may also be an ADHD thing and not really being able to stick
to projects longer term.](conversation://Cadey/coffee)

TL;DR: If you want me to make you useful software, pay me. If you use software
made by others in their spare time and find it useful, pay them. This should not
be a controversial opinion. This should not be a new thing. This should already
be the state of the world and it is amazingly horrible for us to have the people
that make the things that make our software work at all starve and beg for
donations.
