---
title: "GNU Doesn't Care About Your Agency"
date: 2022-02-10
tags:
 - gnu
 - libre
 - rant
---

<div class="warning">

[EDIT(2022-02-10 12:47 EST): I apparently misread part of the GNU #guix channel
rules and made an unreasonable assumption that violators of the rules could be
banned. I have amended a conversation fragment accordingly. My intent was not to
lie, but to point out that some users actually need stuff that nonguix provides
but they just have to know that it exists in the first place.](conversation://Cadey/coffee)

</div>

Or: Ubuntu gives the user more agency about how they want to use their computer
than fully libre GNU/Linux distros ever can.

There are many different kinds of Linux distributions, but today we're going to
think about a certain kind of distribution: ones where the distribution is
totally comprised of free software as much as possible.

These distributions aim to let users benefit by making it possible to study,
hack at and modify every byte of software on the machine's hard drive. This is a
fairly noble goal, however in the process of doing this they break core parts of
hardware compatibility by "de-blobbing" the kernel. Most of these distributions
have a very paternalistic implementation where the "de-blobbed" linux-libre
kernel is the _only_ option, thus limiting users' agency.

For example, let's think about the CPU that I'm using right now. The CPU I'm
using is designed to be able to load CPU microcode updates that are distributed
by the manufacturer in order to mitigate bugs in the microcode that released
with the CPU that can cause real-world impact on what I do. Due to Facts and
Circumstances that are immutable for the sake of argument, this microcode is not
open source and cannot be compiled from source code. The linux-libre kernel
removes the ability to load such firmware updates at runtime.

This means that if something like the FDIV bug or Spectre shows up again but it
can be patched trivially with a microcode update, by nature of using the
linux-libre kernel I am doomed until the base microcode gets updated from the
motherboard manufacturer. If they release a closed-source update that you cannot
inspect or modify.

This paternalistic view of "you shouldn't be able to load microcode updates
because they aren't open source" means that my CPU will be vulnerable to
potentially critical security flaws and I have no way to work around it. This
ends up creating a _limitation_ in how I use my computer. This is worse than the
limitations of proprietary hardware because there is the illusion of free choice
that the community will spout off about as the next coming of sliced bread. That
still doesn't change the fact that my wifi card won't work without the normal
kernel and firmware blobs.

Combine this with other things like wifi card firmware (some wifi cards don't
have the firmware stored on the device, they require the OS to send it firmware
at runtime to make it work at all), and you have actually limited the agency and
capability of users far, far more than if you just let them load the firmware in
the first place.

[Yes, Yes the companies made the hardware this way in the first place and are
responsible for the problem, but telling users they are wrong for wanting it to
work because of an implementation detail about how the hardware updates itself
feels a lot like victim blaming. I am aware of the Talos II being a magical
puppy and rainbow situation where all of this isn't an issue, but sadly the
world just didn't turn out that way and we have to deal with the results of
it.](conversation://Cadey/coffee)

Consider a situation like wanting to play an online game together with friends,
but through Facts and Circumstances you have an Nvidia GPU and the game is on
Steam with no open source option. If you are using a fully open source operating
system with no capacity to install Steam or the Nvidia drivers, you are screwed
and thus your freedom to use your computer how you want is severely limited.

This also extends to how those Linux distributions handle things like AWS. AWS
is largely the poster child of a proprietary cloud hosting platform that you are
made to work with as part of your job. Consider if something like Parabola
GNU/Linux created AWS images and gave users a best-in-class user experience for
using them. This would make the net cost of using a highly auditable environment
a lot lower than the current "don't use AWS lol" (which is again really close to
victim blaming), and would also create institutional knowledge that would let
other people benefit from this as a second or third order effect.

Parabola making AWS images means they can create more generic images, which
means that other people can use those images to do whatever they want with their
own hardware. This lets you have a net benefit to everyone in the project by
decreasing the friction of using it, so it will in turn make users more likely
to adopt it.

Remember the law of halves. Every additional step in adoption costs you half
your audience. Spinning up an AWS instance to mess around with it is a very
low-friction operation.

[But you can just not be a scrub and compile your own traitor kernel that lets
you load freedom-violating binary blobs!](conversation://Numa/delet)

[Then you have to hope your CPU is good enough to build a kernel, hope you can
pay attention to the kernel security mailing list enough to upgrade it when you
need to and finally hope you can upgrade the firmware blobset that the kernel
publishes separately! Hope is not a scalable strategy.](conversation://Cadey/angy)

If their goal is _really_ to liberate users and make it easy for them to have
control over what their computer is doing, they should make it trivial to escape
hatch into a less "pure" setup without having to install third party
repositories that you just have to know about or sidestepping the upstream
update process to install your own system software. This is more victim blaming.

The GNU project could be more than a circlejerk around things that the toe
cheese god said in the 80's and 90's. They could have been a source of reverse
engineering tools, institutions and overall inspire the kind of culture that
would make it _easy_ to understand arbitrary hardware, platforms and software
that you either come across or are made to use as a part of your job.

But they aren't. Instead, Guix, one of their if not their main flagship project
for making a fully GNU system, is addled by the use of the linux-libre kernel.
This makes the kernel fundamentally _incompatible_ with a shocking number of
computers, thus limiting users' freedom to use Guix at all.

[But wait, isn't there that one nonguix project that allows you to install a
normal kernel and Steam?](conversation://Mara/hmm)

[Yeah, but talk about that in the main #guix channel and you get told to not
talk about it. You just have to know that it exists and you can't learn that it
exists without knowing someone that tells you that it exists under the table,
like some kind of underground software drug dealer giving you a hit of wifi card
firmware. This means that knowledge of the nonguix project (which may contain
tools that make it possible to use Guix at all) is hidden from users that may
need it because it allows users to install proprietary software. This limits
user freedom from being able to use their computer how they want by making it a 
potentially untrustable underground software den instead of something that can
be properly handled upstream without having to place trust in too many
places.](conversation://Cadey/angy)

[That hardware is defective by design and you shouldn't use
it.](conversation://Numa/delet)

[Wow, thanks, I'm cured. My wifi card magically stopped existing and now
everything is happy unicorns farting out rainbows that spawn free puppies and
everything is saved forever.<br /><br />Again, that doesn't help me with the
situation that my wifi card doesn't work and I as a user want it to even though
making it work will require proprietary firmware. This shit is how you get
things like the "GPL condom" in the Purism Librem phone, where all the
proprietary firmware is rigged to be loaded automagically in hardware instead of
sofware. This limits your ability to tinker with or modify the firmware _even if
there are legitimate reasons such as critical updates_. So by making the
hardware work with fully free software you have limited the ability to actually
improve the state of the world even with the proprietary firmware the
manufacturer gives you.](conversation://Cadey/angy)

Ubuntu gives the user more agency about how they want to use their computer than
fully libre GNU/Linux distros ever can.
