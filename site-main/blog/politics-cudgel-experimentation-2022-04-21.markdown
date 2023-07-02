---
title: Stop Using Politics As A Cudgel To Discourage Experimentation
date: 2022-04-21
tags:
 - rant
 - systemd
 - communityhealth
---

So let's say you get bored one day and you decide you want to do things that god
and man have decreed impossible. Let's also say that this exact thing involves a
tool that just happens to rustle all of the jimmies (for reasons that are not
entirely clear). Then you get it all to a point where you want to submit it
upstream so you can get help experimenting with this tool.

So you submit it to upstream in the experimental branch, expecting very little
pushback so you can get help tinkering with things. But once you submit it
upstream, [all hell breaks
loose](https://gitlab.alpinelinux.org/alpine/aports/-/merge_requests/33329).

Stop using politics as a cudgel to discourage experimentation. Yes it involves
systemd. Just because you think that the tool is overcomplicated doesn't mean
that other people don't find it useful. Trying to shut down experimentation is
how you get people to leave the community or give up participating in open
source altogether.

The reactions in that thread are both disappointing and somewhat to be expected.
I don't know why people have such a negative reaction to systemd. It's just an
init system, not a religion. It wouldn't have become a good choice for so much
of the Linux ecosystem without it having solid technical merits. If it is really
that bad then the mantle of responsibility is on you for coming up with a better
option.

[No, OpenRC is not that option. It can be PART OF an option, but it is not a
competitor by itself.](conversation://Cadey/coffee)

I know I said I'd stop ranting on this blog as much, but really this stuff
grinds my gears and I feel that I should use my platform for good in this
regard. This is inexcusable. I want to reiterate that I have _no_ power in this
regard. I am just some random person on a blog that got frustrated at the
reactions to this contribution. Some pushback is acceptable. Accusing a
contributor of ignorance is inexcusable. Comments like this have no place in
open source contributions:

> SysTemD is the STD of operating systems. There is no "one little poke", you
> can't be a little bit pregnant.

Jake, if you're out there reading this: keep doing this thing. It is a fantastic
creation that I thought was impossible. You may have to soft-fork the
distribution to get this to work reliably, but I really want to see where this
rabbit-hole goes.

Keep hacking.
