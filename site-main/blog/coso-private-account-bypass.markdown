---
title: "OVE-20221101-0001: counter.social \"private\" account bypass"
date: 2022-12-01
tags:
 - security
 - CoSo
 - mastodon
 - infosec
author: sephiraloveboo
series: CVE
---

<xeblog-hero ai="Waifu Diffusion v1.3 (float16)" file="coso-demon" prompt="jester, spraypaint, graffiti, unlocked, security, 1girl, Taco Bell, touhou, bubble tea, green hair, red eyes, heterochromia, angel wings, kanji inscription"></xeblog-hero>

Incorrect configuration on counter.social allowed random people on the internet
to stalk counter.social users without having an account. Included are numerous
methods people could use to bypass the "private" account system to stalk
counter.social users without having to authenticate. There is also a paid
account feature bypass that allowed any user to trivially create a user account
token with the API and then have the same privilege as the web client. This
normally requires a paid account, but a client that chooses to opt-out of the
security measures didn't require a paid account.

At the time of publication, all of these issues have been patched. 

## Background

[counter.social](https://counter.social) is a social network built on the open
source software [Mastodon](https://joinmastodon.org). For various reasons,
counter.social is one of the few Mastodon servers that **does not** federate to
the larger community, and as such has implemented unique account security
features that allows it to differentiate itself from other Mastodon instances.

The focus today is on the "private" account system. This is a unique account
security feature not implemented in Mastodon itself. This allows users to have
their accounts only visible to other counter.social users and not the wider
internet at large. This feature leaves a lot to be desired. It seems to be
grafted on after the fact using JavaScript instead of integrated into Mastodon's
Ruby on Rails configuration directly. This opens up "private" accounts to
numerous security and privacy issues.

Incidentally, the paid account perk system (for perks like using a custom
Mastodon app) is implemented on the back of the "private" account system. This
also means that the paid account system is easily bypassed by using the same
tricks.

Arguably these are both different issues, but I am tracking them both using the
identifier OVE-20221101-0001 because they rely on the same security mechanism
being bypassed. I attempted to get a CVE ID for this, but I was not able to in
time for publication due to the counter.social modifications being closed
source. If I get a CVE ID, this will be changed accordingly.

## All "private" account logic is done in client-side JavaScript

In general, the entire "private" account system could be bypassed by disabling
JavaScript in the browser, or using a browser that does not have JavaScript
support. This is a trivial change that attackers can enable in their browser.
Alternatively they can configure a content blocker such as µBlock to block this
route:

https://counter.social/authchecker/authchecker.php

Doing so will _completely bypass_ the "private" account system. This
implementation opens users up to their "private" account being publicly visible
through no fault of their own, as the client has to _opt into respecting it_
instead of that feature being baked into the core of counter.social. Mitigation
of this issue would require a _complete rewrite_ of the "private" account system
logic to embed it into Mastodon properly as a Rack middleware instead of
something grafted in after the fact.

Alternatively, you can disable client-side JavaScript execution entirely and get
the same result.

## Visiting a "private" account's URL shows details about the account

Normally when you view a profile page for a user with a "private" account, your
browser is instantly redirected to the page that complains about the user having
a "private" account. This is intended to prevent passive scraping of
counter.social user information. However, this is implemented in such a way that
_all the user information is present_ on the page that generates the redirect.
Using the `curl` command, an untrusted actor from the internet can passively
scrape the HTML of user accounts like this:

```
curl https://counter.social/@th3j35t3r
```

This exposes all of the recent toots made by that user to the public internet,
which is not intended by my understanding. To mitigate this issue, I suggest
changing the implementation of "private" accounts to handle the redirect
_before_ the HTML is rendered.

## Security misconfiguration of ActivityPub "outbox" routes

ActivityPub (the federation protocol Mastodon uses) works by having an "inbox"
and an "outbox". The "inbox" is what other servers post signed messages to and
the "outbox" is what other servers subscribe to. The problem is that all
counter.social users have their outboxes publicly visible. This allows a
malicious actor to view the contents of a counter.social user's posts while
having a "private" profile. For example, here is the outbox for th3j35t3r, who
has a "private" profile:

https://counter.social/users/th3j35t3r/outbox?page=true

It is easy to imagine how this could be problematic, this means an
unsophisticated threat actor could passively scrape "private" profiles for
keywords. To mitigate this issue, I suggest blocking access to the outbox for
unauthenticated users. counter.social is not supposed to be federating anyways.
I suspect it is safe to block this without too much issue.

## Improper security of toots for "private" profiles

On a similar vein to how the "private" account system is implemented, it is
possible for unauthenticated actors to get the contents of individual toots if
they have the URL. Consider this toot by th3j35t3r:

https://counter.social/@th3j35t3r/109265112830539302

This will correctly prevent users from seeing the contents of the toot, because
it is from a "private" account and the user did not opt into public profile
display. However, if you convert the route to this form, it is visible via JSON:

https://counter.social/@th3j35t3r/109265112830539302.json

I suggest requiring authentication for this route or blocking it entirely. This
works because Ruby on Rails (the framework Mastodon is based upon) will
automatically create these handlers for resources when either the URL ends in
.json or the `Accept` header is set to `application/json`. It may be worth
reviewing the Rails configuration and revising things accordingly. This behavior
is endemic to Rails apps and is a core part of how federation works, but this is
not relevant for counter.social because it is not federated. In a pinch, setting
Mastodon's ["secure
mode"](https://docs.joinmastodon.org/admin/config/#authorized_fetch) will block
this for most users.

<xeblog-conv name="Mara" mood="hacker">It's worth noting that this `Accept:
application/json` trick works on most other Mastodon, Akkoma, and Pleroma
servers too. This is how the [toot
embedding](https://xeiaso.net/blog/site-update-mastodon-quoting) feature of this
blog works!</xeblog-conv>

## "Private" accounts can have their profile information viewed publicly

In a similar vein to the above disclosure, it is trivial for an unauthenticated
user to scrape profile information for "private" accounts. You can reformat a
user's profile URL to this form:

https://counter.social/users/th3j35t3r.json

Or simply append `.json` to the end of a user's profile URL:

https://counter.social/@th3j35t3r.json

This also works if you set the `Accept` header to `application/json`.

This route should either be blocked for unauthenticated users or removed
entirely. Setting Mastodon's ["secure
mode"](https://docs.joinmastodon.org/admin/config/#authorized_fetch) will block
this for unauthenticated users.

## Creating a bot token is trivial

Mastodon has a very [rich and featureful
API](https://docs.joinmastodon.org/api/guidelines/). One of the major features
that you can do with the API is authenticate to Mastodon with a username and
password. counter.social prides itself on being free of bots and also requires
users to pay for a subscription in order to use a custom client (such as a bot
API client).

This is trivial to bypass by invoking authentication manually using the [token
grant OAuth2 route](https://docs.joinmastodon.org/methods/oauth/#token) with the
undocumented grant type `password`. The flow for an attacker would look like
this:

* Sign up for an account on counter.social
* Use the [app create](https://docs.joinmastodon.org/methods/apps/#create) call
  to create a new client ID/client secret pair (set to `${COSO_CLIENT_ID}` and
  `${COSO_CLIENT_SECRET}`)
* Construct an HTTP request with the moral equivalent of this curl command:

```shell
curl \
  -F grant_type=password \
  -F client_id=${COSO_CLIENT_ID} \
  -F client_secret=${COSO_CLIENT_SECRET} \
  -F username=azurediamond@itsonlystarsto.me \
  -F password=hunter2 \
```

The `access_token` field in the resulting JSON response can be used to make
requests against the Mastodon API. This can allow a malicious actor to create a
bot with the privileges of any user. Basic stealth methods being employed (such
as only lurking and never posting beyond an introduction message) means that an
attacker could easily slip under the radar and monitor counter.social users
however much they want. This means that counter.social is not free of bots like
it claims and it is impossible to know if there are any existing bots.

Incidentally, this also bypasses part of the paid account upsell system, as you
need a paid account to use a custom client such as Tusky or Toot. Most Mastodon
client apps use this client API, so unless you want to block access to all third
party clients you need to allow this. I am unsure what to suggest here other
than further hardening the authentication logic and checking OAuth2 client IDs
against known good entries.

<xeblog-conv name="Mara" mood="hacker">This works because using the API to
authenticate with a username and password doesn't load HTML into a browser like
the normal OAuth2 flow does. When all of your security can be opted out of by
the client then you don't really have security. You have obscurity. I suspect
that configuring a content blocker for the account validation route would
accomplish the same thing. At the very least, blocking JavaScript works
too.</xeblog-conv>

## Conclusion

The above bypasses for counter.social "private" accounts are sufficient to allow
anyone to anonymously follow counter.social users, read the contents of
individual toots, and view profile information, all without requiring a
counter.social account. I am certain that none of these are intentional. It is
unfortunate that these issues are deep enough that they will require significant
time and energy to mitigate, especially in the wake of Twitter dying.

## Update History

* M11 01 2022: Document was drafted and sent to th3j35t3r to alert him of these
  numerous issues.
* M11 21 2022: Minor wording tweaks were made and a paid account bypass issue
  was revealed.
* M11 23 2022: All issues were confirmed to be patched.
* M12 01 2022: Vulnerability information released to the public.
