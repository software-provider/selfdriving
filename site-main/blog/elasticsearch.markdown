---
title: "The ElasticSearch Rant"
date: 2023-06-10
tags:
  - devops
  - madness
  - go
  - javascript
  - rant
---

<xeblog-hero file="volcano-waifu" ai="SCMix" prompt="volcano, hellfire, burning, fire, 1girl, light green hair, dark green eyes, hoodie, denim, long hair, portrait, masterpiece, best quality, high quality, absurdres, tarot, detailed background"></xeblog-hero>

<xeblog-conv name="Mara" mood="hacker">This post is a rant. Do not take this as
seriously as you would other posts.</xeblog-conv>

As a part of my continued efforts to heal, one of the things I've been trying to
do is avoid being overly negative and venomous about technology. I don't want to
be angry when I write things on this blog. I don't want to be known as someone
who is venomous and hateful. This is why I've been disavowing my
[articles about the V programming language](/blog/series/v) among other things.

ElasticSearch makes it difficult for me to keep this streak up.

I have never had the misfortune of using technology that has worn down my sanity
and made me feel like I was fundamentally wrong about my understanding about
computer science like I have with ElasticSearch. Not since Kubernetes, and I
totally disavow using Kubernetes now at advice of my therapist. Maybe
ElasticSearch is actually this bad, but I'm so close to the end that I feel like
I have no choice but to keep progressing forward.

This post outlines all of my suffering getting ElasticSearch working for
something at work. I have never seen suffering quite as much as this and I am
now two months into a two week project. I have to work on this on fridays so
that I'm not pissed off and angry at computers for the rest of the week.

Buckle up.

<details>
  <summary>For doublerye to read only</summary>
<xeblog-conv name="Cadey" mood="coffee" standalone>Hi, manager! Surely you are reading this post. This post is not an admission of defeat. This post should not be construed as anything but a list of everything that I've had to suffer through to get search working. I'm sorry this has taken so long but I can only do so much when the tools are lying to me.</xeblog-conv>
</details>

## Limbo

ElasticSearch is a database I guess, but the main thing it's used for is as a
search engine. The basic idea of a search engine is to be a central aggregation
point where you can feed in a bunch of documents (such as blogposts or knowledge
base entries) and then let users search for words in those documents. As it
turns out, most of the markdown front matter that is present in most markdown
deployments combined with the rest of the body reduced to plain text is good
enough to feed in as a corpus for the search machine.

However, there was a slight problem: I wanted to index documents from two
projects and they use different dialects of Markdown. Markdown is about as
specified as the POSIX standard and one of the dialects was
[MDX](https://mdxjs.com/), a tool that lets you mix Markdown and React
components. At first I thought this was horrifying and awful, but eventually
I've turned around on it and think that MDX is actually pretty convenient in
practice. The other dialect was whatever [Hugo](https://gohugo.io/) uses, but
specifically with a bunch of custom shortcodes that I had to parse and replace.

<xeblog-conv name="Mara" mood="hacker">Explaining the joke: the POSIX standard
has a lot of core behavior that is formally defined as "implementation defined"
or casually known as "undefined behavior". This does not make it easy to have
programs do fancy things and be portable, so most of the time people copy the
semantics of other POSIX oses just to make life easier.</xeblog-conv>

Turns out writing the parser that was able to scrape out the important bits was
easy, and I had that working within a few hours of hacking thanks to judicious
abuse of line-by-line file reading.

<xeblog-conv name="Cadey" mood="coffee">Sorry for the code maintenance of that!
It was my only real option.</xeblog-conv>

However, this worked and I was at a point where I was happy with the JSON
objects that I was producing. Now, we can get to the real fun: actually using
ElasticSearch.

## Lust

When we first wanted to implement search, we were gonna use something else
(maybe Sonic), but we eventually realized that ElasticSearch was the correct
option for us. I was horrified because I have only ever had bad experiences with
it. I was assured that most of the issues were with running it on your own
hardware and that using Elastic Cloud was the better option. We were also very
lucky at work because we had someone from the DevRel team at Elastic on the
team. I was told that there was this neat feature named AppSearch that would
automatically crawl and index everything for us, so I didn't need to write that
hacky code at all.

So we set up AppSearch and it actually worked pretty well at first. We didn't
have to care and AppSearch dilligently scraped over all of the entries, adding
them to ElasticSearch without us having to think. This was one of the few parts
of this process where everything went fine and things were overall very
convenient.

After being shown how to make raw queries to ElasticSearch with the Kibana
developer tools UI (which unironically is an amazing tool for doing funky crap
with ElasticSearch), I felt like I could get things set up easily. I was feeling
hopeful.

## Gluttony

Then I tried to start using the Go library for ElasticSearch. I'm going to paste
one of the Go snippets I wrote for this, this is for the part of the indexing
process where you write objects to ElasticSearch.

```go
data, err := json.Marshal(esEntry)
if err != nil {
	log.Fatalf("failed to marshal entry: %v", err)
}

resp, err := es.Index("site-search-kb", bytes.NewBuffer(data), es.Index.WithDocumentID(entry.ID))
if err != nil {
	log.Fatal(err)
}

switch resp.StatusCode {
case http.StatusOK, http.StatusCreated:
	// ok
default:
	log.Fatalf("failed to index entry %q: %s", entry.Title, resp.String())
}
```

To make things more clear here: I am using the ElasticSearch API bindings from
Elastic. This is the code you have to write with it. You have to feed it raw
JSON bytes (which I found out was the literal document body after a lot of
fruitless searching through the documentation, I'll get back to the
documentation later) as an io.Reader (in this case a bytes.Buffer wrapping the
byte slice of JSON). This was not documented in the Go code. I had to figure
this out by searching GitHub for that exact function name.

<xeblog-conv name="Aoi" mood="wut">Wait, you're using an API wrapper. Why do you
need to check the HTTP status code manually? Those status codes are documented
somewhere, right?</xeblog-conv>
<xeblog-conv name="Cadey" mood="percussive-maintenance">Oh dear, bless your
sweet and innocent heart. The API client only really handles authentication and
randomly trying queries between several nodes in a cluster. It doesn't handle
raising errors on unsuccessful HTTP status codes. God forbid you have to read
anything out of the reply, because you have to parse the JSON
yourself.</xeblog-conv>
<xeblog-conv name="Aoi" mood="coffee">How does this program exist.</xeblog-conv>

Oh yeah, I forgot to mention this but they don't ship error types for
ElasticSearch errors, so you have to json-to-go them yourself. I really wish
they shipped the types for this and handled that for you. Holy cow.

## Greed

Now you may wonder why I went through that process if AppSearch was working well
for us. It was automatically indexing everything and it should have been fine.
No. It was not fine, but the reason it's not fine is very subtle and takes a
moment to really think through.

In general, you can break most webpages down into three basic parts:

- The header which usually includes navigation links and the site name
- The article contents (such as this unhinged rant about ElasticSearch)
- The footer which usally includes legal overhead and less frequently used
  navigation links (such as linking to social media).

When you are indexing things, you usually want to index that middle segment,
which will usually account for the bulk of the page's contents. There's many
ways to do this, but the most common is the
[Readability](https://www.npmjs.com/package/@mozilla/readability) algorithm
which extracts out the "signal" of a page. If you use the reader view in Firefox
and Safari, it uses something like that to break the text free from its HTML
prison.

So, knowing this, it seems reasonable that AppSearch would do this, right? It
doesn't make sense to index the site name and navigation links because that
would mean that searching for the term "Tailscale" would get you utterly useless
search results.

Guess what AppSearch does?

<xeblog-conv standalone name="Aoi" mood="facepalm">Oh god, how does this just
keep getting worse every time you mention anything? Who hurt the people working
on this?</xeblog-conv>

Even more fun, you'd assume this would be configurable. It's not. The UI has a
lot of configuration options but this seemingly obvious configuration option
wasn't a part of it. I can only imagine how sites use this in practice. Do they
just return article HTML when the AppSearch user-agent is used? How would you
even do that?

I didn't want to figure this out. So we decided to index things manually.

## Anger

So at this point, let's assume that there's documents in ElasticSearch from the
whole AppSearch thing. We knew we weren't gonna use AppSearch, but I felt like I
wanted to have some façade of progress to not feel like I had been slipping into
insanity. I decided to try to search things in ElasticSearch because even though
the stuff in AppSearch was not ideal, there _were_ documents in ElasticSearch
and I could search for them. Most of the programs that I see using ElasticSearch
have a fairly common query syntax system that lets you write searches like this:

```
author:Xe DevRel
```

And then you can search for articles written by Xe about DevRel. I thought that
this would rougly be the case with how you query ElasticSearch.

Turns out this is nowhere near the truth. You actually search by POST-ing a JSON
document to the ElasticSearch server and reading back a JSON document with the
responses. This is a bit strange to me, but I guess this means that you'd have
to implement your own search DSL (which probably explains why all of those
search DSLs vaguely felt like special snowflakes). The other main problem is how
ElasticSearch uses JSON.

To be fair, the way that ElasticSearch uses JSON probably makes sense in Java or
JavaScript, but not in Go. In Go
[`encoding/json`](https://pkg.go.dev/encoding/json) expects every JSON field to
only have one type. In basically every other API I've seen, it's easy to handle
this because most responses really do have one field mean one thing. There's
rare exceptions like message queues or event buses where you have to dip into
[`json.RawMessage`](https://pkg.go.dev/encoding/json#RawMessage) to use JSON
documents as containers for other JSON documents, but overall it's usually fine.

ElasticSearch is not one of these cases. You can have a mix of things where
simplified things are empty objects (which is possible but annoying to
synthesize in Go), but then you have to add potentially dynamic child values.

<xeblog-conv name="Mara" mood="hacker" standalone>To be fair there is a typed
variant of the ElasticSearch client for Go that does attempt to polish over most
of the badness, but they tried to make it very generic in ways that just
fundamentally don't work in Go. I'm pretty sure the same type-level tricks would
work a lot better in Rust.</xeblog-conv>

Go's type system is insufficiently typeful to handle the unrestrained madness
that is ElasticSearch JSON.

When I was writing my search querying code, I tried to use mholt's excellent
[json-to-go](https://mholt.github.io/json-to-go/) which attempts to convert
arbitrary JSON documents into Go types. This did work, but the moment we needed
to customize things it became a process of "convert it to Go, shuffle the output
a bit, and then hope things would turn out okay". This is fine for the Go people
on our team, but the local ElasticSearch expert wasn't a Go person.

<xeblog-conv name="Cadey" mood="coffee" standalone>If you are said ElasticSearch
expert that has worked with me throughout this whole debacle, please don't take
my complaining about this all as a slight against you. You have been the main
reason that I haven't given up on this project and when this thing gets shipped
I am going to make every effort to highlight your support throughout this. I
couldn't have made it this far without you.</xeblog-conv>

Then my manager suggested something as a joke. He suggested using
[`text/template`](https://pkg.go.dev/text/template) to generate the correct JSON
for querying ElasticSearch. It worked. We were both amazed. The even more cursed
thing about it was how I quoted the string.

In `text/template`, you can set variables like this:

```
{{ $foo := <expression> }}
```

And these values can be the results of arbitrary template expressions, such as
the `printf` function:

```
{{ $q := printf "%q" .Query }}
```

The thing that makes me laugh about this is that the grammar of `%q` in Go is
enough like the grammar for JSON strings that I don't really have to care
(unless people start piping invalid Unicode into it, in which case things will
probably fall over and the client code will fall back to suggesting people
search via Google). This is serendipidous, but very convenient for my usecase.

If it ever becomes an issue, I'll probably encode the string to JSON with
[`json.Marshal`](https://pkg.go.dev/encoding/json#Marshal), cast it to a string,
and then have that be the thing passed to the template. I don't think it will
matter until we have articles with emoji or Hanzi in them, which doesn't seem
likely any time soon.

<xeblog-conv standalone name="Aoi" mood="wut">I guess this would also be really
good for making things maintainable such that a non-Go expert can maintain it
too. At first this seemed like a totally cursed thing, but somehow I guess it's
a reasonable idea? Have I just been exposed to way too much horror to think this
is terrible?</xeblog-conv>

## Treachery

Another fun thing that I was running into when I was getting all this set up is
that seemingly all of the authentication options for ElasticSearch are broken in
ways that defy understanding. The basic code samples tell you to get
authentication credentials from Elastic Cloud and use that in your program in
order to authenticate.

This does not work. Don't try doing this. This will fail and you will spend
hours ripping your hair out because the documentation is lying to you. I am
unaware of any situation where the credentials in Elastic Cloud will let you
contact ElasticSearch servers and overall this was really frustrating to find
out the hard way. The credentials in Elastic Cloud are for programmatically
spending up more instances in Elastic Cloud.

<xeblog-conv name="Aoi" mood="coffee" standalone>Why isn't any of this
documented?</xeblog-conv>

ElasticSeach also has a system for you to get an API key to make requests
against the service so that you can use the principle of least privilege to
limit access accordingly. This doesn't work. What you actually need to do is use
username and password authentication. This is the thing that works.

<xeblog-conv name="Cadey" mood="coffee" standalone>To be honest, I felt like
giving up at this point. Using a username and password was a thing that we could
do, but that felt _wrong_ in a way that is difficult for me to describe. I was
exceedingly happy that _anything_ worked and it felt like I was crawling up a
treadmill to steal forward millimeters at a time while the floor beneath me was
pulling away meters at a time. This is when I started to wonder if working on a
project like this to have "impact" for advancing my career was really worth the
sanity cost.</xeblog-conv>

Then I found out that the ping API call is a privileged operation and you need
administrative permissions to use it. Same with the "whoami" call.

<xeblog-conv name="Aoi" mood="rage">Okay, how the fuck does that make any sense
at all? Isn't it a fairly common pattern to do a "check if the server is working
and the authentication is being accepted appropriately" call before you try to
do anything with the service? Why would you protect that? Why isn't that a
permissionless call? What about the "whoami" call? Why would you need to lock
that down behind any permissions? This just makes no sense. I don't get it.
Maybe I'm missing something here, maybe there's some fundamental aspect about
database design that I'm missing. Am I missing anything here or am I just
incompetent?</xeblog-conv>
<xeblog-conv name="Numa" mood="happy">Welcome to the CLUB!</xeblog-conv>
<xeblog-conv name="Aoi" mood="facepalm">This never ends, does it?</xeblog-conv>
<xeblog-conv name="Numa" mood="happy">Nope. Welcome to hell.</xeblog-conv>

Really though, that last string of "am I missing something fundamental about how
database design works or am I incompetent?" thoughts is one of the main things
that has been really fucking with me throughout this entire saga. I try to not
let things that I work on really bother me (mostly so I can sleep well at
night), but this whole debacle has been very antithetical to that goal.

## Blasphemy

Once we got things in production in a testing capacity (after probably spending
a bit of my sanity that I won't get back), we noticed that random HTTP responses
were returning 503 errors without a response body. This bubbled up weirdly for
people and ended up with the frontend code failing in a weird way (turns out it
didn't always suggest searching Google when things fail). After a bit of
searching, I think I found out what was going on and it made me sad. But to
understand why, let's talk about what ElasticSearch does when it returns fields
from a document.

In theory, any attribute in an ElasticSearch document can have one or more
values. Consider this hypothetical JSON document:

```json
{
  "id": "https://xeiaso.net/blog/elasticsearch",
  "slug": "blog/elasticsearch",
  "body_content": "I've had nightmares that are less bad than this shit",
  "tags": ["rant", "philosophy", "elasticsearch"]
}
```

If you index such a document into ElasticSearch and do a query, it'll show up in
your response like this:

```json
{
  "id": ["https://xeiaso.net/blog/elasticsearch"],
  "slug": ["blog/elasticsearch"],
  "body_content": ["I've had nightmares that are less bad than this shit"],
  "tags": ["rant", "philosophy", "elasticsearch"]
}
```

And at some level, this really makes sense and is one of the few places where
ElasticSearch is making a sensible decision when it comes to presenting user
input back to the user. It makes sense to put everything into a string array.

However in Go this is really inconvenient and if you run into a situation where
you search "`NixOS`" and the highlighter doesn't return any values for the
article (even though it really should because the article is _about_ NixOS), you
can get a case where there's somehow no highlighted portion. Then because you
assumed that it would always return something (let's be fair: this is a
reasonable assumption), you try to index the 0th element of an empty array and
it panics and crashes at runtime.

<xeblog-conv name="Aoi" mood="wut" standalone>Oh, wait, is this why Rust makes
Vec indexing return an `Option<T>` instead of panicking if the index is out of
bounds like Go does? That design choice was confusing to me, but it makes a lot
more sense now.</xeblog-conv>

We were really lucky that "`NixOS`" was one of the terms that did this behavior,
otherwise I suspect we would never have found it. I did a little hack where it'd
return the first 150 characters of an article instead of a highlighted portion
if no highlighted portion could be found (I don't know why this would be the
case but we're rolling with it I guess) and that seems to work fine...until we
start using Hanzi/emoji in articles and we end up cutting a family in half.
We'll deal with that when we need to I guess.

## Thievery

While I was hacking at this, I kept hearing mentions that the TypeScript client
was a lot better, mostly due to TypeScript's type system being so damn flexible.
You can do the
[N-Queens problem](https://lemoine-benoit.medium.com/eight-queens-puzzle-in-typescripts-type-system-4c7ff1693545)
in the type solver alone!

However, this is not the case and for some of it it's not really Elastic's fault
that the _entire JavaScript ecosystem_ is garbage right now. As for why:
consider this code:

```javascript
import * as dotenv from "dotenv";
```

This will blow up and fail if you try to run it with `ts-node`. Why? It's
because it's using
[ECMAScript Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
instead of "classic" CommonJS imports. This means that you have to _transpile_
your code from the code you wish you could write (using the `import` keyword) to
the code you have to write (using the `require` function) in order to run it.

Or you can do what I did and just give up and use CommonJS imports:

```javascript
const dotenv = require("dotenv");
```

That works too, unless you try to import an ECMAScript module, then you have to
use the `import` _function_ in an async function context, but the top level
isn't an async context, so you have to do something like:

```javascript
(async () => {
  const foo = await import("foo");
})();
```

This does work, but it is a huge pain to not be able to use the standard import
syntax that you should be using anyways (and in many cases, the rest of your
project is probably already using this standard import syntax).

<xeblog-conv name="Mara" mood="hacker" standalone>This is one of the cases where
[Deno](https://deno.land) gets things right, Deno uses ECMAScript modules by
default. Why can't everything else just do this? So frustrating.</xeblog-conv>

Anyways, once you get past the undefined authentication semantics again, you can
get to the point where your client is ready to poke the server. Then you take a
look at
[the documentation for creating an index](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html#_create_2).

<xeblog-conv standalone name="Aoi" mood="wut">Seriously, what is it with these
ElasticSearch clients and not having strictly defined authentication semantics,
I don't get it.</xeblog-conv>

In case Elastic has fixed it, I have recreated the contents of the
`indicies.create` function below:

> <big>create</big>
>
> Creates an index with optional settings and mappings.
>
> [Endpoint documentation](https://www.elastic.co/guide/en/elasticsearch/reference/8.8/indices-create-index.html)
>
> ```
> client.indices.create(...)
> ```

Upon seeing this, I almost signed off of work for the day. What are the function
arguments? What can you put in the function? Presumably it takes a JSON object
of some kind, but what keys can you put in the object? Is this library like the
Go one where it's thin wrappers around the raw HTTP API? How do JSON fields
bubble up into HTTP request parts?

Turns out that there's a whole lot of conventions in the TypeScript client that
I totally missed because I was looking right at the documentation for the
function I wanted to look at. Every method call takes a JSON object that has a
bunch of conventions for how JSON fields map to HTTP request parts, and I missed
it because that's not mentioned in the documentation for the method I want to
read about.

Actually wait, that's apparently a lie because
[the documentation](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/introduction.html)
doesn't actually spell out what the conventions are.

<xeblog-conv name="Cadey" mood="coffee" standalone>I realize that my
documentation consumption strategy may be a bit backwards here. I usually start
from the immediate thing I'm trying to do and then work backwards from there to
my solution. A code sample for the function in question _at the place where the
function is documented_ would have helped a lot.</xeblog-conv>

I had to use ChatGPT as a debugging tool of last resort in order to get things
working at all. To my astonishment, the suggestions that ChatGPT made worked. I
have never seen anything documented as poorly as this and I thought that the
documentation for NixOS was bad.

<xeblog-sticker name="Cadey" mood="percussive-maintenance"></xeblog-sticker>

## Fin

If your documentation is bad, your user experience is bad. Companies are usually
cursed to recreate copies of their communication structures in their products,
and with the way the Elastic documentation is laid out I have to wonder if there
is any communication at all inside there.

One of my coworkers was talking about her experience trying to join Elastic as a
documentation writer and apparently part of the hiring test was to build
something using ElasticSearch. Bless her heart, but this person in particular is
not a programmer. This isn't a bad thing, it's perfectly reasonable to not
expect people with different skillsets to be that cross-functional, but good
lord if I'm having this much trouble doing _basic operations with the tool_ I
can't expect anyone else to really be able to do it without a lot of
hand-holding. That coworker asked if it was a Kobayashi Maru situation (for the
zoomers in my readership: this is an intentionally set up no-win scenario
designed to test how you handle making all the correct decisions and still
losing), and apparently it was not.

Any sufficiently bad recruiting process is indistinguishable from hazing.

I am so close to the end with all of this, I thought that I would put off
finalizing and posting this to my blog until I was completely done with the
project, but I'm 2 months into a 2 week project now. From what I hear,
apparently I got ElasticSearch stuff working rather quickly (???) and I just
don't really know how people are expected to use this. I had an ElasticSearch
expert on my side and we regularly ran into issues with _basic product
functionality_ that made me start to question how Elastic is successful at all.

I guess the fact that ElasticSearch is the most flexible option on the market
helps. When you start to really understand what you can do with it, there's a
lot of really cool things that I don't think I could expect anything else on the
market to realistically accomplish in as much time as it takes to do it with
ElasticSearch.

ElasticSearch is just such a huge pain in the ass that it's making me ultimately
wonder if it's really worth using and supporting as a technology.
