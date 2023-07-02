---
title: Writing Coherently At Scale
date: 2022-06-29
tags:
 - writing
vod:
  youtube: https://youtu.be/pDOoqqu06-8
  twitch: https://www.twitch.tv/videos/1513874389
---

As someone who does a lot of writing, I have been asked how to write about
things. I have been asked about it enough that I am documenting this here so you
can all understand my process. This is not a prescriptive system that you must
do in order to make Quality Content™️, this is what I do.

<xeblog-conv name="Cadey" mood="coffee">I honestly have no idea if this is a
"correct" way of doing things, but it seems to work well enough. Especially so
if you are reading this.</xeblog-conv>

<xeblog-hero file="great-wave-cyberpunk" prompt="the great wave off of kanagawa, cyberpunk, hanzi inscription"></xeblog-hero>

## The Planning Phase

To start out the process of writing about something, I usually like to start
with the end goal in mind. If I am writing about an event or technology thing,
I'll start out with a goal that looks something like this:

> Explain a split DNS setup and its advantages and weaknesses so that people can
> make more informed decisions about their technical setups.

It doesn't have to be very complicated or intricate. Most of the complexity
comes up naturally during the process of writing the intermediate steps. Think
about the end goal or what you want people to gain from reading the article. 

I've also found it helps to think about the target audience and assumed skills
of the reader. I'll usually list out the kind of person that would benefit from
this the most and how it will help them. Here's an example:

> The reader is assumed to have some context about what DNS is and wants to help
> make their production environment more secure, but isn't totally clear on how
> it helps and what tradeoffs are made.

State what the reader is to you and how the post benefits them. Underthink it.
It's tempting to overthink this, but really don't. You can overthink the
explanations later.

### The Outline

Once I have an end goal and the target audience in mind, then I make an outline
of what I want the post to contain. This outline will have top level items for
generic parts of the article or major concepts/steps and then I will go in and
add more detail inside each top level item. Here is an example set of top level
items for that split DNS post:

```markdown
- Introduction
- Define split DNS
- How split DNS is different
- Where you can use split DNS
- Advantages of split DNS
- Tradeoffs of split DNS
- Conclusion
```

Each step should build on the last and help you reach towards the end goal. 

After I write the top level outline, I start drilling down into more detail. As
I drill down into more detail about a thing, the bullet points get nested
deeper, but when topics change then I go down a line. Here's an example:

```markdown
- Introduction
  - What is DNS?
    - Domain Name Service
    - Maps names to IP addresses
      - Sometimes it does other things, but we're not worrying about that today
    - Distributed system
      - Intended to have the same data everywhere in the world
      - It can take time for records to be usable from everywhere
```

Then I will go in and start filling in the bullet tree with links and references
to each major concept or other opinions that people have had about the topic.
For example:

```markdown
- Introduction
  - What is DNS?
    - Domain Name Service
      - https://datatracker.ietf.org/doc/html/rfc1035
    - Maps names to IP addresses
      - Sometimes it does other things, but we're not worrying about that today
    - Distributed system
      - Intended to have the same data everywhere in the world
      - It can take time for records to be usable from everywhere
        - https://jvns.ca/blog/2021/12/06/dns-doesn-t-propagate/
```

These help me write about the topic and give me links to add to the post so that
people can understand more if they want to. You should spend most of your time
writing the outline. The rest is really just restating the outline in sentences.

## Writing The Post

After each top level item is fleshed out enough, I usually pick somewhere to
start and add some space after a top level item. Then I just start writing. Each
top level item usually maps to a few paragraphs / a section of the post. I
usually like to have each section have its own little goal / context to it so
that readers start out from not understanding something and end up understanding
it better. Here's an example:

> If you have used a computer in the last few decades or so, you have probably
> used the Domain Name Service (DNS). DNS maps human-readable names (like
> `google.com`) to machine-readable IP addresses (like `182.48.247.12`). Because
> of this, DNS is one of the bedrock protocols of the modern internet and it
> usually is the cause of most failures in big companies.
> 
> DNS is a globally distributed system without any authentication or way to
> ensure that only authorized parties can query IP addresses for specific domain
> names. As a consequence of this, this means that anyone can get the IP address
> of a given server if they have the DNS name for it. This also means that
> updating a DNS record can take a nontrivial amount of time to be visible from
> everywhere in the world.
> 
> Instead of using public DNS records for internal services, you can set up a
> split DNS configuration so that you run an internal DNS server that has your
> internal service IP addresses obscured away from the public internet. This
> means that attackers can't get their hands on the IP addresses of your
> services so that they are harder to attack. In this article, I'm going to
> spell out how this works, the advantages of this setup, the tradeoffs made in
> the process and how you can implement something like this for yourself.

In the process of writing, I will find gaps in the outline and just fix it by
writing more words than the outline suggested. This is okay, and somewhat
normal. Go with the flow.

I expand each major thing into its component paragraphs and will break things up
into sections with markdown headers if there is a huge change in topics. Adding
section breaks can also help people stay engaged with the post. Giant walls of
text are hard to read and can make people lose focus easily. 

Another trick I use to avoid my posts being giant walls of text is what I call
"conversation snippets". These look like this:

<xeblog-conv name="Mara" mood="hacker">These are words and I am saying
them!</xeblog-conv>

I use them for both creating [Socratic
dialogue](https://en.wikipedia.org/wiki/Socratic_dialogue) and to add prose
flair to my writing. I am more of a prose writer [by
nature](https://xeiaso.net/blog/the-oasis), and I find that this mix allows me
to keep both technical and artistic writing up to snuff.

<xeblog-conv name="Cadey" mood="enby">Amusingly, I get asked if the characters
in my blog are separate people all giving their input into things. They are
characters, nothing more. If you ever got an impression otherwise, then I have
done my job as a writer _incredibly well_.</xeblog-conv>

Just flesh things out and progressively delete parts of the outline as you go.
It gets easier.

### Writing The Conclusion

I have to admit, I really suck at writing conclusions. They are annoying for me
to write because I usually don't know what to put there. Sometimes I won't even
write a conclusion at all and just end the article there. This doesn't always
work though.

A lot of the time when I am describing how to do things I will end the article
with a "call to action". This is a few sentences that encourages the reader to
try the thing that I've been writing about out for themselves. If I was turning
that split DNS article from earlier into a full article, the conclusion
could look something like this:

> ---
>
> If you want an easy way to try out a split DNS configuration, install
> [Tailscale](https://tailscale.com/) on a couple virtual machines and enable
> [MagicDNS](https://tailscale.com/kb/1081/magicdns/). This will set up a split
> DNS configuration with a domain that won't resolve globally, such as
> `hostname.example.com.beta.tailscale.net`, or just `hostname` for short.
> 
> I use this in my own infrastructure constantly. It has gotten to the point
> where I regularly forget that Tailscale is involved at all, and become
> surprised when I can't just access machines by name.
> 
> A split DNS setup isn't a security feature (if anything, it's more of an
> obscurity feature), but you can use it to help administrate your systems by
> making your life easier. You can update records on your own schedule and you
> don't have to worry about outside attackers getting the IP addresses of your
> services.

I don't like giving the conclusion a heading, so I'll usually use a [horizontal
rule (`---` or `<hr
/>`)](https://www.coffeecup.com/help/articles/what-is-a-horizontal-rule/) to
break it off.

---

This is how I write about things. Do you have a topic in mind that you have
wanted to write about for a while? Try this system out! If you get something
that you like and want feedback on how to make it shine, email me at
`iwroteanarticle at xeserv dot us` with either a link to it or the draft
somehow. I'll be sure to read it and reply back with both what I liked and some
advice on how to make it even better.
