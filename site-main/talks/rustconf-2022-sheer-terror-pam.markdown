---
title: The Sheer Terror of PAM
date: 2022-09-05
slides_link: https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slides.pdf
---

Hey all, this is my [RustConf 2022](https://rustconf.com/) for 2022! I'm super
excited to _finally_ be able to share this page publicly. Enjoy the talk! I've
included a video of it, my script and slides embedded below and finally a link
to the slides PDF at the bottom of the page. Choose how you want to enjoy this
talk. All the options are vaild.

<xeblog-conv name="Cadey" mood="enby">Be well all!</xeblog-conv>

---

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/BleS-MHLn7s" title="RustConf 2022 - THE SHEER TERROR OF PAM by Xe Laso" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></center>

![The introduction slide](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.001.jpeg)

Hi, I'm Xe Iaso and today I'm going to get to talk about existential terror. I'm
going to talk about The Sheer Terror of PAM, the authentication and
authorization framework for Linux and UNIX systems. By the end of this talk you
should have a clear understanding of what PAM is, why it exists, why it sucks,
and the horrors I encountered while writing a PAM module in Rust for work.

![My about the speaker
slide](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.002.jpeg)

Like I said, I'm Xe Iaso. I'm a blogger, student of philosophy, and an aspiring
fiction writer. Professionally I work at Tailscale as the Archmage of
Infrastructure doing developer relations stuff. It's worth noting that I'm not
speaking for Tailscale in this talk, but I am speaking about a project I worked
on at Tailscale. Any opinions are my own.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.003.jpeg)

Authentication and authorization are two similar sounding concepts, but they are
subtly different in ways that really set them apart. So that everyone's on the
same page, here's my SRE cliffs notes version of the terms.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.004.jpeg)

Authentication is what makes sure you are who or what you say you are. This is
what we use passwords, spicy hardware dongles that pretend to be a keyboard, and
Google Authenticator codes for. This only proves the identity of a person or
service though.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.005.jpeg)

Authorization is what makes sure that you are allowed to do something. This is
why you get those annoying user access control prompts on windows or asked to
scan your face to install apps on an iPad.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.006.jpeg)

The difference between the two is that authentication would be Kirby confirming
who he is as the reincarnation of the god of death and authorization would be
making sure Kirby has permission to recreate the world in his image. He is
authenticated as the reincarnation of Dark Matter but he would not be authorized
to use Star Dream to reshape Planet Popstar to his will.

<xeblog-conv name="Cadey" mood="coffee">Sorry for the major spoilers for every
Kirby game there. Kirby lore. Quite something.</xeblog-conv>

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.007.jpeg)

So, now that we know what authentication and authorization are, let's talk about
the one library that everyone across most Linux distributions, Unices and Macs
use for this: PAM.

PAM is an authentication and authorization framework for Linux and UNIX systems.
It's not really one implementation, but they're all similar enough that we can
pretend they are one big happy implementation just to keep things simple.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.008.jpeg)

The biggest thing that PAM does that the things that came before it doesn't are
that PAM allowed administrators to set up policies for how people could connect
to computers. Today this means you can set up rules that match your buzzword
compliant policies.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.009.jpeg)

PAM is the result of the 90's making the computing world a lot more complicated
when multiple computers got into the mix. PAM came around after people realized
that storing password hashes for everyone in the system wouldn't work very well
if you made that file with all the password hashes world-readable.

Computer security was a vastly different thing when the whole world ran on
paper, eh?

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.010.jpeg)

PAM was made by the Java people: Sun Microsystems. PAM was intended to help
replace other competing implementations of tacky directory server integration
(like shoving all of the password database into DNS) and get rid of the existing
legacy cruft that developed organically over the years.

It is kind of amusing that PAM in all its incarnations is now the organically
grown cruft that it was set out to replace.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.011.jpeg)

But computers got faster at crunching numbers and with that storing more
important classified work, so something had to give. Imagine how bad it would
have been if hackers could grab your password hash over DNS and send it to a
cluster of 4 whole PDP-11 minicomputers to crack it.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.012.jpeg)

PAM ships with a bunch of modules that let you check passwords against the local
password database, run arbitrary commands on the system, check for account
expiration or password aging, and change your password.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.013.jpeg)

By the way, that "changing your password" thing was something that I had a
misconception about for a while when I was learning this. When I read the manual
pages out of Ubuntu, I thought that password modules were for customizing what
password checking rules were in place. I admit I made an assumption on
incomplete information, this stuff is hard to research.

Turns out that password modules in PAM are for changing your password, not for
changing the authentication logic for using passwords. The FreeBSD handbook
helped clear that up for me. FreeBSD has amazing documentation by the way, check
it out sometime.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.014.jpeg)

At the time C was the best thing since canned bread, so PAM was written in C.
Each PAM module was implemented as its own shared library file also written in
C.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.015.jpeg)

And this worked for the time that it was made for. It met all the needs and it
had extensibility by both systems administrators and software distributors. It
worked great for its time.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.016.jpeg)

However it's not the 90's anymore. It's 2022. Password theft is rampant. The
computer in most display controllers on fancy laptops can rival the computers
that PAM was designed for. Two factor auth is mandatory for securing access to
production. And to top it all off, somehow society has agreed that OAuth2 and
OIDC are decent standards. C has aged about as gracefully as milk in a desert,
which makes it really easy to write arbitrary code execution bugs that also do
something useful sometimes.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.017.jpeg)

But we still have PAM in every Mac, Ubuntu server and all over the cloud. We
will probably still have to deal with PAM for a very long time. It's been around
for about 25 years now and the longer something is around the longer it'll take
to get rid of it. However, it's so ubiquitous that you can take advantage of
this to make your servers do whatever you want on login.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.018.jpeg)

One of the really cool features of PAM is that every service on the system can
have its own rules. When an application starts an authentication session with
PAM, it supplies a service name. This service name is used to enable
administrators to set up policies for that service in particular.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.019.jpeg)

At a high level, each service has a stack of configuration rules. Each of these
rules tells PAM which one of the modules it should use and how PAM should react
to the module's return codes. It also lets you jump around the stack and then go
towards whatever you want.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.020.jpeg)

Hmm, kind of reminds me of a certain language construct. Can't place my finger
on what it is. Hmmm.

<xeblog-conv name="Numa" mood="delet">goto!</xeblog-conv>

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.021.jpeg)

There is an upside to this though, combined with service-specific configuration
this means you can set rules like "logging in with a password also requires you
to use a Google Authenticator code" or "when a login succeeds, log it to Slack
or Discord", or even "if the user gives you a password, reject the
authentication session". With this you can make it difficult to do the wrong
thing. It's just annoying to debug.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.022.jpeg)

You may be wondering something like "how do you debug this?"

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.023.jpeg)

Conveniently, PAM has logging calls all over the place. But additionally
conveniently they are all removed at compile time in most Linux distribution
packages. 

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.024.jpeg)

So the answer of how you debug stuff in PAM boils down to "carefully". You debug
it carefully. You know what, here's a story of my first time debugging a custom
PAM module.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.025.jpeg)

My goal was to try and make a PAM module for Tailscale. Tailscale is kinda like
a VPN (but not a "privacy" VPN) and one of the neat things about Tailscale IP
addresses is that you have a 1:1 mapping of IP addresses and users. As a result,
this means you can use Tailscale as an authentication layer.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.026.jpeg)

So after some frantic hacking I had this module and from everything I could tell
it should have been working. I had it configured as an authentication module at
the top of the stack, I made it put a "please god be working" message into
syslog, I tried to make it print things to standard out, nothing was working. So
I started googling "how to debug PAM" and got answers on forums from 2002 that
mentioned Classic Red Hat Linux and Slackware as if those were the only two
distributions that exist.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.027.jpeg)

Of course that didn't work, so I needed to get a debugger into the mix. After
looking over the various options at my disposal, GDB seemed the least painful. I
also kind of remembering using it in college. A decade ago.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.028.jpeg)

So I started SSHD in "debug" mode under GDB and then set a breakpoint for pam
underscore sm underscore authenticate. If I set this breakpoint, then surely I
get debugger access to PAM at that point. I started SSHD and tried to connect.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.029.jpeg)

Authentication failed. No breakpoint was triggered.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.030.jpeg)

Nothing useful except GDB happily telling me that SSHD exited with exit
status 0. A successful. exit. code.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.031.jpeg)

So I looked around some more to see if there was any decent documentation. I had
to dig really deep. Really really deep.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.032.jpeg)

I eventually found what I was looking for in a SunOS manual that was so old it
was a HTML frameset in the internet archive. Somehow only the content of the
page was there and the rest of the frameset had degraded. But, I got a bit of C
code that let me manually invoke PAM so that I could test a module manually.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.033.jpeg)

So, it turns out that when you authenticate with PAM, under the hood PAM calls
the dlopen system call on the target modules in the configuration file. dlopen
is a C library call that lets you load a shared library file from the disk into
memory and then call functions in it as if they were functions that were
compiled into the heccin' binary in the first place.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.034.jpeg)

Just like Java. The Sun Microsystems influence had to go somewhere!

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.035.jpeg)

So what I need to do is not set my breakpoint on dlopen, but I need to set it
on a similar function named dlsym. dlsym is like dlopen, but it lets you get the
address of a function out of a dlopen handle and run it.

Based on past experience maintaining a tire fire of a chat server that used
dynamically loaded C modules everywhere, dlsym should normally be called just
before the function gets called. I validated this against the little test
program that the SunOS manual gave me and everything worked. I even got the
syslog calls like I expected. Then I tried it again with SSHD and I got a
breakpoint at dlsym!

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.036.jpeg)

I win! I have a step in the right direction! So I asked gdb for the stack trace
to debug further. And...

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.037.jpeg)

...I get a bunch of useless function pointers. This was confusing but then I
figured out what was going on.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.038.jpeg)

Ubuntu strips debug symbols from binaries in the packages they release to
consumers, which will make stack traces full of hex pointers instead of names.
However you can download the debug packages to get the names back. After
remembering the right apt incantations, I was able to dig through the source
code in parallel to the debugger session. After hacking around for a while I got
nowhere, but in so much detail. I put an SOS message off in a Slack and signed
off for the day.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.039.jpeg)

Next morning I woke up and checked Slack. A coworker had a suggestion and it
broke me a bit. But to understand why, let's look at SSHd again.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.040.jpeg)

Ubuntu's default SSH server is OpenBSD's OpenSSHd. OpenSSHd doesn't ship with
PAM support on by default because the OpenBSD people think that's insecure. I
can't blame them. Yoloing in binaries off the disk, plucking functions out of
them, and jumping to them while hoping they work is not safe at all. So they
added a configuration option called ChallengeResponseAuth that nerfs most of the
fun that PAM can have. It's off by default in Ubuntu. Turning that on made
everything work instantly.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.041.jpeg)

It was that easy. It did everything I wanted, including logging things to
syslog.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.042.jpeg)

With all that in mind, let's talk about the drawbacks of PAM's design that I've
learned.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.043.jpeg)

Most of PAM's safety is predicated on everyone following the rules and doing
things as safe as possible. When a PAM module is loaded into a service, the code
in that module can't assume practically anything about what's going on. It can't
assume what user it's running as, what files it has access to, if it has access
to the network stack, or even if it has anything at all. Much less when
sandboxes and containers are in play.

In practice, basically every PAM service runs either as root or as effectively
root. So this means that you can usually rely on having access to anything that
root can, but assume you don't.

<xeblog-conv name="Mara" mood="happy">This is why I'm calling it "as safe as an
earthquake". If you follow the building code to the letter and do absolutely
everything correctly, you're fine. If you don't, oops the preschool building
made out of asbestos just cracked in two with all the kids hiding in it. Good
luck with that!</xeblog-conv>

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.044.jpeg)

One of my friends is a hardcore C advocate. He has made a bunch of things in C
that nobody should try to make in C, such as web applications. This person also
adamantly claims that C has a type system so it's clearly safe to do that.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.045.jpeg)

Except I can find no evidence of C having a type system outside of it just being
"numbers that the machine knows how to deal with" and "hoping everything goes
into the right places". The epitome of this is the yolo pointer, also known as a
void pointer. A yolo pointer points to some address in RAM. Where is it? Don't
know. Does it contain anything useful? Hope so. How do you know what the data
is? Look at it.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.046.jpeg)

By the way this works for function pointers too. You can make a pointer to a
function out of a void pointer. Interestingly, this is how that Games Done Quick
run of Ocarina of Time where they collected the Triforce worked. Nintendo wrote
all their games in C  at the time and used function pointers all over the place.
So speedrunners just wrote their own function pointers with the control stick
angle and buttons and got code execution.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.047.jpeg)

A wise friend of mine said "hope is not a strategy". Hope is the strategy when
dealing with yolo pointers.

PAM uses these everywhere with "items" associated with the PAM handle. This
means that from a type system level you can put the integer 42 into the "remote
IP address" string field and the compiler will be like "yeah sure whatever". But
at runtime you can get a segfault out of nowhere if someone messed up. Most
people do follow the rules though, so this usually works out.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.048.jpeg)

PAM was designed to have a fairly flexible configuration framework _for systems
administrators_.

But one of the things they did not do at the PAM level is make it easy to have
third parties slipstream PAM modules into the main system without having to have
deep knowledge of the PAM stack and everything it does.

In practice, this means that each distribution has its own PAM configuration
mechanism and base PAM configuration to deal with, and the documentation
standards for how to do that can be poor.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.049.jpeg)

And by "poor" I mean "the only documentation is a design document deep in the
archives of the Ubuntu community wiki". Thankfully this was accurate, but
realizing that the only documentation is a specification is not a good feeling.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.050.jpeg)

When you are linking Rust code into C code, you need to be very sure that you
are not leaking memory in any way. It will never be freed. Rust is usually good
about not letting you leak memory on accident, but there are a few ways you can
leak it on purpose.

<xeblog-conv name="Numa" mood="delet">Don't.</xeblog-conv>

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.051.jpeg)

While I was off on my magic adventure into PAM land, I learned these lessons for
dealing with PAM in the future:

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.052.jpeg)

When you are writing security-critical things like this, it's best to be very
careful about what dependencies you bring in. Keep the list small. If one of
those gets owned, so does production. This can be "bad".

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.053.jpeg)

If you keep the scope of your PAM module small, you don't have to worry about
scope creep. If you set out to do one thing, do that one thing and stop. Don't
overthink this. If you need to do multiple things, write multiple PAM modules.

<iframe width="560" height="315" src="https://www.youtube.com/embed/HjbfjCPuO2Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Less is so much more.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.054.jpeg)

Using PAM in general doesn't let you make very many assumptions about service
configuration stacks. There could be just about anything going on there.

Using PAM on Ubuntu 22.04 lets you make a lot of assumptions about the stack.
Plan for the today that is, not the tomorrow that might be.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.055.jpeg)

In practice, you should not use handwritten bindings to any C library like I did
when I implemented tailpam. Use tools like bindgen. They're there for a reason.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.056.jpeg)

Also make sure that your code is fast. Authentication can happen surprisingly
often. If authentication hangs, that makes people get worried. SRE adrenaline is
a very powerful force. Be careful with it.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.057.jpeg)

In conclusion, PAM seems like a sheer terror at first but it's really just a
weird C library that everyone standardized on. It's fairly easy to extend with
custom logic in C or Rust, and it allows you to do whatever you want with
whatever you want. It's the industry standard, used by small scrappy startups
like Google, Facebook, Amazon, and Apple. And you've used it nearly every time
you've logged into a Linux desktop.

It's kind of terrifying, but at least it's everywhere. If you grab some random
Linux/Unix off the shelf you can likely see PAM in there somewhere. Unless
you're using OpenBSD or something, they describe PAM as "an ugly pile of hacks"
and oh boy I cannot agree more.

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.058.jpeg)

Before I get all of this wrapped up, I just want to take a moment to shout out
everyone on this list for helping make the talk shine. 

Thank you all!

![](https://cdn.xeiaso.net/file/christine-static/blog/rc2022/slide.059.jpeg)

And thank you for watching! I'm going to stick around in the chat for questions,
but if I miss your question somehow and you really want an answer to it, please
email it to rustconf2022 at xeserv dot us.

I will have the written form of this talk up on my website tomorrow, including
my slide deck and everything I've said today.

If you have questions, please speak up and ask them. I am more than happy to
answer them.

Be well, all.

