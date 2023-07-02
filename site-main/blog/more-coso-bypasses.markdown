---
title: "More counter.social \"private account\" bypasses"
date: 2022-12-20
author: ectamorphic
tags:
 - CoSo
 - RubyOnRails
 - hacking
---

<xeblog-hero ai="Waifu Diffusion v1.3" file="apocalypse-waifu" prompt="mushroom cloud, cityscape, 1girl, gas mask, ninja, dystopian"></xeblog-hero>

Hi there. This is a followup to my article about [the vulnerabilities I found in
a mastodon server named
counter.social](https://xeiaso.net/blog/coso-private-account-bypass). This
community is powered by a very hacked up fork of
[Mastodon](https://joinmastodon.org/), a popular federated social media platform
you can self-host that behaves something like Twitter did before the Elon
takeover.

## Background

[counter.social](https://counter.social) is a social network built on the open
source software [Mastodon](https://joinmastodon.org). For various reasons,
counter.social is one of the few Mastodon servers that **does not** federate to
the larger community, and as such has implemented unique account security
features that allows it to differentiate itself from other Mastodon instances.
It also has an embedded stream of CNN and other news sites.

This social network is run by the hacktivist th3j35t3r. He has an [extensive
rapsheet](https://en.wikipedia.org/wiki/The_Jester_(hacktivist)) and had drama
with popular hacking groups like LulzSec. th3j35t3r is a very unstable figure in
the best of times, so it has been interesting to see the fallout of his
operations of a Mastodon server.

<xeblog-conv name="Cadey" mood="coffee">For various reasons, I think that the
best way to describe counter.social's federation policy as "should not federate"
rather than "does not federate". But, I digress, for all practical reasons you
can treat it as "does not federate" because they broke the federation API in
weird ways.</xeblog-conv>

Earlier in November 2022, I discovered a number of _trivial exploits_ that could
let you bypass its "private account" system, also called a "public landing
page". One of the main things this system lets you do is have an account that is
"public" to other users of counter.social, but does not index on Google search.
This security method was implemented using JavaScript and a HTML `<iframe>`
element. Needless to say, this mechanism was trivially bypassed by either using
`curl -H "Accept: application/json"` or disabling JavaScript in your browser.

<xeblog-sticker name="Cadey" mood="facepalm"></xeblog-sticker>

## The fixes

To give th3j35t3r some credit, the main problems I reported were fixed. If you
make unauthenticated requests to the federation API, Cloudflare's web
application filter will block them. If you view those things from a browser, you
get 302 redirected to the same iframe that says "lol no that page doesn't work
friend".

I'd personally like to see the redirect be replaced with either the 404 page or
some error page that describes things instead of the 302 redirect, but this does
work.

The other problems seem to have been fixed by blocking other API calls, but I
don't know if the way that the API calls were blocked would break other clients
working. For various reasons that I am about to get into, this was difficult to
test.

<xeblog-conv name="Mara" mood="hmm">Wait, isn't most of this technically public
information? All of the posts you've mentioned have a privacy profile of
`public`, which means that being able to see them isn't surprising or novel at
all. Why would anyone care about this?</xeblog-conv>

<xeblog-conv name="Cadey" mood="coffee">Here's why:<xeblog-picture path="blog/coso-private-account-blockpage"></xeblog-picture></xeblog-conv>

<xeblog-conv name="Mara" mood="hmm">Yeah, maybe that is a good
reason.</xeblog-conv>

<xeblog-conv name="Cadey" mood="coffee">If I'm not supposed to view a thing
without being authenticated, it's a bug if I'm viewing it without being
authenticated.</xeblog-conv>

## counter.social participates in retaliation against researchers doing responsible disclosure

When I made that earlier disclosure, I was banned in retaliation after making my
standard public announcement on it. I don't have the contents of the toot
anymore, but I said something along the lines of "I hacked counter.social" on
counter.social with my burner account and I was banned within 5 minutes because
of it. After [complaining about this on
Twitter](https://twitter.com/theprincessxena/status/1600160959946887168), I got
blocked on Twitter too.

Amusingly my home IP address is listed as a "rogue state actor", which my
friends at the pretend CIA have to disagree with:

<xeblog-toot url="https://national-defence.network/notice/AQKqw3MT6vRE6MF2O0"></xeblog-toot>

<xeblog-conv name="Cadey" mood="enby">Honestly this is one of the highlights of
my career as a technical communicator!</xeblog-conv>

Honestly, at this point I was ready to just call things a loss and ignore the
situation. Nothing good would come of me raising a stink about things, and
apparently th3j35t3r has been leveling false representations about the
vulnerability I discovered and reported. He got all up in arms about me
accessing "public content", but if it was public then why do users not have the
public landing page feature enabled? Why would that be "public"? It doesn't make
sense, but apparently we're dealing with troll logic so who knows anymore?

This whole situation also frustrates me in particular because I used to look up
to th3j35t3r as a bit of a hacktivist role model when I was in my edgelord
teenager phase that everyone goes through. I am told that finding out your
childhood heroes are shitbags is a common experience and that in general you
should never meet your heroes. I can't agree more after this experience.

Oh also apparently there was supposed to be a bug bounty for counter.social for
doing the first exploit. I did not receive any such payout. Should th3j35t3r be
reading this post, please get in contact with me to arrange payment. I take
PayPal, Bitcoin, and Ethereum.

I found more vulnerabilities in counter.social's "private account" feature. I'm
going to step through them and explain how the routing stack in Rails works so
that I can show you how to find more of them.

<xeblog-conv name="Mara" mood="hacker">As a regular reminder, if you are running
a Mastodon server that you don't want to federate with the public Internet and
additionally don't want it to be open for scraping, you should enable [secure
mode](https://docs.joinmastodon.org/spec/activitypub/#secure-mode) to prevent
unauthorized servers from federating with your server. If th3j35t3r would have
enabled secure mode, none of these vulnerabilities would work.</xeblog-conv>

Here's some irresponsible disclosure.

## The "featured" collection

Every so often I look at the error logs in my servers. This is purely for my own
amusement, and one day I ended up looking at the error log on my
[Akkoma](https://akkoma.social/) server. Akkoma is a Mastodon-compatible social
media server, and as such it will sometimes give you error messages when it has
trouble poking mastodon servers for whatever reason. I noticed a URL pattern
come up a few times:

```
/users/{username}/collections/featured
```

When I ran this against [my Mastodon account](https://pony.social/@cadey), I got
the contents of my pinned posts. This immediately made me wonder if this would
work on `counter.social`. Needless to say, it does. This is yet another
information disclosure that can be _easily solved_ by enabling secure mode. This
route shows you pinned posts so that viewing a new remote profile on another
server shows you something useful other than "lol I don't know anything yet".

<xeblog-conv name="Cadey" mood="coffee">Also for what it's worth, I was able to
access the featured collection on my home IP address even after it was blocked.
I'm starting to get this weird feeling that the IP address blocklist is also
implemented with client-side JavaScript. I haven't cared to reverse-engineer
things though.</xeblog-conv>

I thought to try this on counter.social because I got a random follow from
someone on it to my pony.social account. This struck me as super odd because I
thought that the server wasn't meant to federate. It is still partially
federating but both the follow back and DM I sent are not going to result in
anything useful I don't think. Apparently the sidekiq jobs that I made doing
that are never going to succeed for a very weird reason: th3j35t3r changed the
server inbox route to `/etc/passwd`.

## Security through what the hell

In ActivityPub, actors have inboxes and outboxes. New items are put into the
inbox and fetched from the outbox. Conventionally, each actor's inbox is
supposed to be at `https://{actor_base}/inbox` and the outbox is supposed to be
at `https://{actor_base}/outbox`. Servers in ActivityPub have their own inboxes
and outboxes for various federation errata.

For some reason, th3j35t3r changed the server inbox to
`https://counter.social/etc/passwd`. This breaks incoming federation from remote
ActivityPub servers. I don't have any idea why this path in particular was
chosen, but my Mastodon server admin suspects it's so that they can file abuse
complaints with hosters that people are trying to get the passwords of the
remote system.

<xeblog-conv name="Cadey" mood="coffee">If you have ideas, please let me know
what they are. I have no clue here.</xeblog-conv>

This is an interesting case of security through obscurity, but I feel that it
would be better to just block all federation attempts instead.

## More digging

Since this was the _fifth_ such information disclosure I have found, I started
to wonder if there were more that I wasn't aware of. So like any rational person
I decided to download the source code of Mastodon and take a look at its routing
stack to see what else is there.

```
$ git clone https://github.com/mastodon/mastodon ~/code/mastodon/mastodon
```

When you are looking at a program, it's usually useful to start at the `main`
function and work your way out from there. Most programs will declare everything
the program needs in its `main` function and pass those around to start various
things up. One of the main entrypoints into apps written with [Ruby on
Rails](https://rubyonrails.org/) is its routing stack at `config/routes.rb`. So
let's look
[there](https://github.com/mastodon/mastodon/blob/main/config/routes.rb).

Some things that stand out are [the two routes that I used to attack
counter.social in the
past](https://github.com/mastodon/mastodon/blob/c1de6730604f526a6c2d19adcf6f195352de0641/config/routes.rb#L84-L85).
These routes let you fetch [a user as
json](https://pony.social/users/cadey.json) or [one of their posts as
json](https://pony.social/users/cadey/statuses/109542758692635818.json).

<xeblog-conv name="Mara" mood="hacker">When other servers are federating, they
don't just blindly append `.json` to the end of URLs like these examples here
do. These exmaple URLs tack on `.json` to force Rails to render JSON to the user
to help illustrate the point. In practice federated requests will have
authentication via signatures and use the `Accept: application/json`
header.</xeblog-conv>

Then we get to the [meaty
part](https://github.com/mastodon/mastodon/blob/c1de6730604f526a6c2d19adcf6f195352de0641/config/routes.rb#L88-L108).
This table of routes spells out all of the things that clients can do over HTTP.
I'm not totally sure how that relates to HTTP urls, or routes, but we can easily
start comparing the route names in here against the [ActivityPub
spec](https://www.w3.org/TR/activitypub/) and [Mastodon
documentation](https://docs.joinmastodon.org/spec/activitypub/).

The collections thing stands out. It seems that there are three kinds of
collections defined in the code:

- featured - seems to be pinned posts
- devices - not sure what this is
- tags - seems to be set to the featured tags on my profile on pony.social

Only the featured collection seems to work on counter.social. Other routes of
interest are the outbox (which seems to be incorrectly blocked as you can access
the root outbox path that tells you how many toots a counter.social user made,
which may be useful for other kinds of signal intelligence), his list of
followers (which 403s), and the list of replies (which also 403s).

The only information disclosure left is the featured collection.

## Webfinger

ActivityPub relies on [Webfinger](https://docs.joinmastodon.org/spec/webfinger/)
to look up information on remote users. Webfinger operates on
`/.well-known/webfinger`, and if you block that route, federation will just
totally fail.

Guess what counter.social doesn't block? This lets you get metadata about users
and any aliases they have. This is ultimately useless from my standpoint, but
blocking that _would remove most of the problems to begin with_, so there is that.

## Somebody set up us the bomb

Fun fact: you can put [my Mastodon profile into your RSS
reader](https://pony.social/users/cadey.rss). Another fun fact: this works on
counter.social too and lets you bypass the authorization logic for the "private
account" feature. This also works if you add `.rss` to the end of the URL too,
and it might work if you add `.atom` to the end of the URL.

Have fun! 
