---
title: I Was Part of a Human Subject Research Study Without My Consent
date: 2021-12-17
author: ectamorph
---

[Note for the readers, I usually try to do one post per week. This is not that
post this week. I am just frustrated by being used as a human subject in a
Princeton study without my consent. If you are ever in a position to be doing
this kind of survey, please don't send legal threats around
recklessly.](conversation://Cadey/coffee)

This is a response to [CCPA Scam November
2021](https://blog.freeradical.zone/post/ccpa-scam-2021-12/) from the
[freeradical.zone](https://blog.freeradical.zone/) blog. On or about 2021-12-11
11:29 PM I got an email from [Maya Mishina](mailto:mayamishina@novatormail.ru)
with the following contents:

> To Whom It May Concern:
> 
> My name is Maya Mishina, and I am a resident of Novosibirsk, Russia. I have a
> few questions about your process for responding to California Consumer Privacy 
> Act (CCPA) data access requests:
>
> 1. Would you process a CCPA data access request from me even though I am not a
>    resident of California?
> 2. Do you process CCPA data access requests via email, a website, or
>    telephone? If via a website, what is the URL I should go to?
> 3. What personal information do I have to submit for you to verify and process
>    a CCPA data access request?
> 4. What information do you provide in response to a CCPA data access request?
>
> To be clear, I am not submitting a data access request at this time. My
> questions are about your process for when I do submit a request.
> 
> Thank you in advance for your answers to these questions. If there is a better
> contact for processing CCPA requests regarding christine.website, I kindly ask
> that you forward my request to them.
> 
> I look forward to your reply without undue delay and at most within 45 days of
> this email, as required by Section 1798.130 of the California Civil Code.
> 
> Sincerely,
>
> Maya Mishina

This scared the shit out of me. My blog is a passion project that I do as a way
to get better at writing. I almost contacted a lawyer. This should probably have
stood out as suspect to me, however I am a believer that people have a right to
privacy and that they should be able to be forgotten. 

I go out of my way to ensure that this website handles as little user data as
possible. I have gone so far to do this that the only unique identifiers I deal
with are IP addresses, but even then only a tiny fraction of those IP addresses
even get to my server because I use Cloudflare for caching. I probably need to
set up some kind of proper log rotation in my server, but right now there are
only three things collected by my site:

1. If you are a patron to my site, your name shows up on [/patrons](/patrons).
   This queries the Patreon API to get the display name that you configured in
   your account and is refreshed every time the site restarts.
2. If you send me a [Webmention](https://en.wikipedia.org/wiki/Webmention), they
   will be shown on the footer of the website. They are stored in a SQLite
   database on the same server. I can remove entries from that table upon
   request.
3. Your IP address is recorded in `/var/log/nginx/xesite.access.log` in common
   log format on my server which is located in the Netherlands. Here is an
   example of what this looks like:
   
```
127.0.0.1 - - [18/Dec/2021:04:04:57 +0000] "GET /security.txt HTTP/1.1" 404 2110 "-" "Go-http-client/2.0"
```

No other data is stored. Any intermediate things in the system journal disappear
after an hour or two because my journal is limited to 512 MB and my services are
chatty.

Here is what the application logged for that request:

```
Dec 18 04:04:57 lufta xesite-start[2121878]: 2021-12-18T04:04:57.585717Z  INFO xesite/2.3.0: - "GET /security.txt HTTP/1.1" 404 "-" "Go-http-client/2.0" 12.474µs
```

It is literally the bare minimum that I can get away with.

[Wanna know why there's no comments on this blog? I don't want to have to deal
with storing user data and doing moderation!](conversation://Cadey/coffee)

I probably should have consulted a lawyer before drafting this, but here is what
I replied with:

> Hello,
>
> 1. I can process a CCPA data access request even for people that are not
>    residents of California.
> 2. My website does not collect personal information, but emailing either
>    me@christine.website or privacy@xeserv.us would be the correct action.
> 3. My website does not collect personal information, including IP
>    addresses. I keep track of hit counts via CloudFlare analytics, but as
>    far as I know there is no way for me to collect information about a
>    single subject (all I see is aggregate anonymized data). I guess if you
>    give me a public IP address I can dig through the system logs to see if
>    anything pops up.
>    4. I would provide relevant request logs provided they exist. That is
>    the only information that I could provide and I would be willing to
>    provide it in the industry standard plaintext log format.
> 
> Please keep in mind this is a blog run by a single person as a passion
> project to get better at writing. As it is not a corporate endeavor, I
> don't believe that I need to provide this information at all. However I
> am willing to search the logs folder to see if anything is there.
> 
> Please let me know which IP addresses to look up and I will do my best
> to get you that information as fast as possible.
>
> Thanks and be well,
>
> Xe

I should have expected this to be a human research study after the [University
of Minnesota
disaster](https://www.theverge.com/2021/4/30/22410164/linux-kernel-university-of-minnesota-banned-open-source).

Overall, I am disappointed in this. I want to have a positive outlook on
humanity. I want to be able to trust that requests to be forgotten are
legitimate. I am going to have a harder time doing that now.

In case you actually do want to make a GDPR/CCPA request, here is the process
and the rough steps I will take:

- I will need your Twitter username and any links listed on the Webmentions
  section of any article on my blog if you want those removed. I will require
  you to either tweet a magic phrase and email me a link to it, a DNS TXT
  record on the domain or for you to upload an arbitrary string to a path on
  your server. These processes will help me confirm that you are the person who
  wants the information. Send me an email to `privacy@xeserv.us`. If you want
  access logs deleted, please let me know what IP addresses to delete and I
  will see what I can do.
- I will prepare a `.zip` file that will contain the following:
   - The email conversation in the industry-standard `.eml` format.
   - Any SQLite rows from my Webmention service in `.csv` format.
   - Any request logs from nginx in `.log` format.
- This may take up to a week or two. I am only one person and I have a job.
  This blog is not my primary source of income.

I do not have to offer this service. I am not a business. I am a single person
that wants to get better at writing. Please do not include wording that gives me
the impression that you are making a legal threat. I know you are allowed to,
but it scares the shit out of me when you do that and will make me put your
request at the end of the list.

tl;dr: I got phished by trying to be a good netizen. Don't fall for [this
scam](https://privacystudy.cs.princeton.edu/).
