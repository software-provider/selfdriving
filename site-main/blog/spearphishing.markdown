---
title: "Spearphishing: it can happen to you too"
date: 2022-07-09
tags:
 - linkedin
 - infosec
---

<xeblog-hero file="the-fool" prompt="The Fool in a woodcut tarot card style"></xeblog-hero>

For some reason, LinkedIn has become the de-facto social network for
professionals. It is viewed as a powerful networking and marketing site that
lets professionals communicate, find new opportunities and source talent at
eye-watering speed and rates. However, at the same time this also means that
LinkedIn becomes a treasure trove of data to enable spearphising attacks.

Let's consider [this attack against popular "play to earn" game Axie
Infinity](https://www.theblock.co/post/156038/how-a-fake-job-offer-took-down-the-worlds-most-popular-crypto-game).
The attackers had PDF based malware that allowed them to get access to a target
computer, so they needed someone to open a PDF to trigger the exploit chain that
let them gain a foothold. But they specifically wanted people that likely had
access to the crypto wallets that enable control of the blockchain. LinkedIn let
them filter by employees at the company behind Axie Infinity that were
developers and likely started spearphishing by role and seniority. The details
of the attack spell out that the attackers had set up a whole fake interview
process to convince the marks that the process was legitimate and they put the
malware in the offer letter. The attackers later gained access to the validator
wallets and then they were able to make off with over half a billion dollars
worth of cryptocurrency.

<xeblog-conv name="Numa" mood="delet">Maybe, just maybe you shouldn't store a
majority of the keys required to validate something on _the same computer_.
Especially if those keypairs control assets worth close to _half a billion
dollars_. Holy heck.</xeblog-conv>

The malware was in the offer letter. This is the kind of social engineering
attack that I bet any one of you reading this article could fall for. Hell, I'd
probably fall for this. This may be the wrong kind of take to have, but I'm
really starting to wonder if using LinkedIn so much is actually bad for
security. It's not just recruiters reading through LinkedIn anymore, it's also
threat actors that are trying to break in and do God knows what. Maybe we as an
industry should stop feeding all of that data into LinkedIn. Not only would it
give you less recruiter spam, maybe it'll make spearphishing attacks more
difficult too.

<xeblog-conv name="Cadey" mood="coffee">Also, yes we can't trust PDFs anymore,
especially after exploits like
[FORCEDENTRY](https://googleprojectzero.blogspot.com/2021/12/a-deep-dive-into-nso-zero-click.html)
became a thing.</xeblog-conv>

Either way, I may end up getting a disposable machine for dealing with reading
PDFs from unknown sources in the future. I could use a virtual machine for this,
but if my threat model includes PDFs having exploits in them then I probably
can't trust a virtual machine to be a reasonable security barrier. I don't know.
It sucks that we can't trust people anymore.

I kinda wish we could.

---

<xeblog-conv name="Mara" mood="hacker">Fun fact: the tarot card "The Fool"
doesn't actually imply idiocy in a malicious way. The major arcana of the tarot
is a bunch of memes that describe the story of The Fool's journey through magick
and learning how the world works. The Fool is not an idiot, The Fool is just
someone that is unaware of the difficulties they are going to face in life and
treats things optimistically. Think a free spirit as opposed to someone that is
foolhardy (though foolhardiness is the meaning of The Fool when the card is
inverted).</xeblog-conv>
