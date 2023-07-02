---
title: "Site Update: Version 3.0"
date: 2022-11-26
author: Heartmender
series: site-update
tags:
 - dhall
 - LaTeX
 - rust
 - typescript
 - Xeact
---

<xeblog-hero ai="Waifu Diffusion v1.3 (float16)" file="aoi-onsen" prompt="1girl, fox ears, dark blue hair, blue eyes, kimono, festival, landscape, makoto shinrai, arknights, fireworks, pagodas, onsen, long hair, princess, -hands, -amputee"></xeblog-hero>

Hey all! Welcome to Xesite 3.0! Over the last few days I've taken a huge chunk
out of my backlog and I have redone _a lot_ of this website. My hope is that
this will make it faster, more reliable and ultimately make things a lot easier
for everyone. There's a lot of improvements here so I'm just going to start
going over them one by one.

## Project name changed

This website has historically been named `site`. It's had a lot of work put into
it over the years and I've never really referred to it by anything but "the
website" or "my blog". However I've noticed a change in my notes and I think I
should make this official. The project behind this website is now officially
called `xesite`.

<xeblog-conv name="Mara" mood="hmm">What about all of the custom HTML tags?
Aren't they all called `xeblog-$NAME`? Are you going to change
those?</xeblog-conv>

<xeblog-conv name="Cadey" mood="coffee">The custom HTML tags are an
implementation detail, not something that is exposed to end users. Plus, those
are now in my muscle memory so I can't really change those if I wanted
to.</xeblog-conv>

## Blogpost series have metadata

When I implemented series support in the blog, it was kind of a hack. Series are
intended to function kind of like tags, but more for tagging things that
progress in a logical series. As an example, take a look at the page for my
[site-to-site WireGuard VPN](https://xeiaso.net/blog/series/site-to-site-wireguard)
series. It shows a little description for the series and in the future I will
likely add other metadata like a series image.

This is powered by my new `SeriesDescription` Dhall record type.

<xeblog-conv name="Mara" mood="hacker">For context,
[Dhall](https://dhall-lang.org) is a Haskell-like non-Turing-complete
configuration language that you can think of like JSON or YAML but with
functions, imports, and static types.</xeblog-conv>

I've never really covered how Dhall works on this blog before, so I'll take a
look at this and the `seriesDescriptions.dhall` file to show off how damn cool
Dhall is.

One of the most unique features of Dhall as a configuration language is its
record completion operator `::`. This allows you to specify a record (read:
object)'s type and also a set of _default values_ for that record. Let's take a
look at the `SeriesDescription` record type:

```dhall
-- dhall/types/SeriesDescription.dhall
{ Type = { name : Text, details : Text }
, default = { name = "", details = "" }
}
```

If I wanted to use this from inside my Dhall configuration, I would need to:

* Import the type
* Create a value with the `::` operator

The description for the `site-to-site-wireguard` series could look something
like this:

```dhall
-- dhall/series/site-to-site-wireguard.dhall
let xesite = ../types/package.dhall

let SeriesDescription = xesite.SeriesDescription

in SeriesDescription::{
   , name = "site-to-site-wireguard"
   , details = "Instructions on setting up your own VPN with WireGuard."
   }
```

You declare your imports with URLs or filesystem paths and other aliases using
`let`, and then you declare what to do with those variables after `in`. If
you've never used Haskell or Lisp before, you can imagine `let` creating a block
of scope just like an extra layer of curly braces in JavaScript or other C-like
languages:

```javascript
{
  let foo = "bar";
  console.log(foo);
}

// foo isn't usable from here
```

As you can imagine, this gets [way more
elaborate](https://github.com/Xe/site/blob/main/dhall/signalboost.dhall) when
you get more detail into the mix. However, it's all really easy to understand.
It's just different than you might be expecting at first glance.

If you've never really used Dhall before, I really suggest taking a look at it.
It's what I use for the configuration of all of my tools when I'm allowed to
make those decisions. It's also got the ability to read from environment
variables, which can make it easier to safely turn a bunch of raw strings from
the environment into a more sensible datatype.

## New type for the Signal Boost data file

One of my most successful parts of this site is the [signal boost
page](https://xeiaso.net/signalboost). It was the first place I used Dhall on
this website, and is one of the most contributed to things I've ever had in one
of my own projects.

When I designed the schema forever ago, I apparently decided that people should
only have one of a few types of links. I thought a Twitter and GitHub link
should be sufficient. The Dhall type used to look something like this:

```dhall
{ Type =
    { name : Text
    , tags : List Text
    , gitLink : Optional Text
    , twitter : Optional Text
    , linkedin : Optional Text
    , fediverse : Optional Text
    , coverLetter : Optional Text
    , website : Optional Text
    }
, default =
  { name = ""
  , tags = [] : List Text
  , gitLink = None Text
  , twitter = None Text
  , linkedin = None Text
  , fediverse = None Text
  , coverLetter = None Text
  , website = None Text
  }
}
```

I was wrong. That is _not_ generic enough. Recently I got [this pull
request](https://github.com/Xe/site/pull/567/files) that made me really rethink
how I'm doing this. I don't want to limit people to a set number of links and
then deal with that, I want things to be generic enough that people can link to
whatever they want. I ended up settling on this:

```dhall
let Link = ./Link.dhall

in  { Type = { name : Text, tags : List Text, links : List Link.Type }
    , default =
      { name = "", tags = [] : List Text, links = [] : List Link.Type }
    }
```

This lets people link to _whatever they want_, such as like this:

```dhall
-- dhall/signalboost/XeIaso.dhall
let xesite = ../types/package.dhall

let Link = xesite.Link

let Person = xesite.Person

in  Person::{
    , name = "Xe Iaso"
    , tags =
      [ "Go"
      , "Rust"
      , "Dhall"
      , "Nix"
      , "NixOS"
      ]
    , links =
      [ Link::{ url = "https://github.com/Xe", title = "GitHub" }
      , Link::{ url = "https://xeiaso.net", title = "Blog" }
      ]
    }
```

You can easily imagine how this would make things a lot simpler in practice.
Instead of having to have specific fields for every kind of link someone could
have, there's a list of links with a custom target and title. I hope this will
make things simpler in practice.

I have also removed a lot of the older entries with people that I have
determined to be employed. If I am mistaken, please submit a pull request with
corrections.

## Resume is now a PDF

I've hosted [my resume](https://xeiaso.net/resume) on my blog for years. It's
been a Markdown file (originally generated from a JSON document back when JSON
resume was a thing) for the longest time. Whenever someone wanted a copy of my
resume, I usually shoved that markdown file into Pandoc and gave them the
resulting PDF. This has not scaled.

Recently I changed how [my salary transpareny page
works](https://xeiaso.net/blog/site-update-salary-transparency) and in the
process I put all of my job history information (including salaries) into a
giant heckin' Dhall document. It was always my intention to come back around and
use this data for the resume page, but I've been stuck on the exact best way to
do this.

On Wednesday I had a bad idea. By the end of the day Thursday, I had it working.
One of the features Dhall has is a string interpolation operator. If you write a
document like this:

```dhall
let bar = "bar"
in  "foo${bar}"
```

Dhall will return `foobar`. This means that you can use Dhall for both
structured data _and text templating_. I've also been maintaining a separate
immigration-friendly resume for a few years using Overleaf and LaTeX (turns out
border agencies don't like it when you use a professional pseudonym on your
resume, who knew). The bad idea was to shove the template into Dhall somehow so
that Dhall would puke out LaTeX source code which would then be fed into a LaTeX
compiler to turn into a PDF.

Of course, if I was going to do this, I couldn't just choose pdfTeX as my LaTeX
compiler of choice. That's not on-brand enough. It just so turns out that there
is an international-character friendly LaTeX compiler named
[XeTeX](https://en.wikipedia.org/wiki/XeTeX). I'm not kidding. It is so on-brand
of me to use this that it _hurts_.

So I took all of the madness in my trollish heart and poured it into my emacs
frame. I eventually ended up with
[`resume.dhall`](https://github.com/Xe/site/blob/v3/dhall/latex/resume.dhall).
When you run `dhall text` on that file, you get all of that data transformed
into LaTeX source code [like
this](https://gist.github.com/Xe/59eccf3750697aba51512e571d971207).

However having a bunch of LaTeX source code isn't useful by itself (unless you
are also an immortal and can read LaTeX like I can). I wanted the resume to
build itself so I _cannot forget how this works_. When thinking about build
systems that are largely that ignorable, I usually turn to
[Nix](https://nixos.org). So I wrote a Nix package for building my resume and
shoved it into the resulting package for my blog. It looks something like this:

```nix
# tex is texlive-medium plus a bunch of packages I need

resumePDF = pkgs.stdenv.mkDerivation {
  pname = "xesite-resume-pdf";
  inherit (bin) version;
  inherit src;
  buildInputs = with pkgs; [ dhall tex ];

  phases = "installPhase";

  installPhase = ''
    mkdir -p $out/static/resume
    cp -rf ${pkgs.dhallPackages.Prelude}/.cache .cache
    chmod -R u+w .cache
    export XDG_CACHE_HOME=.cache
    export DHALL_PRELUDE=${pkgs.dhallPackages.Prelude}/binary.dhall;

    ln -s $src/dhall/latex/resume.cls
    dhall text --file $src/dhall/latex/resume.dhall > resume.tex

    xelatex ./resume.tex
    cp resume.pdf $out/static/resume/resume.pdf
  '';
};
```

Now if I want to build my resume by itself, I can run this command:

```console
$ nix build .#resumePDF
```

This should make it a lot easier to deal with recruiters!

## I rewrote basically all of the templates with Maud

When I ported my website from [Go to
Rust](https://xeiaso.net/blog/site-update-2020-07-16) back in 2020 I needed a
library like Go's [html/template](https://pkg.go.dev/html/template) to template
out the HTML that my site uses. At the time there were many options I could pick
from, but I ended up choosing [ructe](https://github.com/kaj/ructe) because it
would compile the templates into my application binary instead of having to ship
those with my website. This also means that the _optimizer_ can chew through my
templates and make them _even faster_ than html/template. Native code will
_always_ be faster than interpreted code.

This worked for a while, but I started running into ergonomics problems as I
continued to use ructe. The great part about ructe is that because the templates
are compiled to Rust anyways, you can use any Rust logic or types you want. The
horrible part about ructe is that your editor autocomplete and type checking
logic doesn't work. Debugging compile failures of your templates requires that
you understand how the generated code works. This isn't really as much of an
issue as I'm making it sound like, but it's a papercut nonetheless.

A while ago I found the Rust library [Maud](https://maud.lambda.xyz/). It's a
procedural macro that uses a Rust-like syntax to emit HTML. It kinda looks like
HTML if you squint. Compare these two identical documents:

```html
<p>Hi there this is a test!</p>
```

```rust
html! {
  p { "Hi there this is a test!" }
}
```

It also lets you use [splices](https://maud.lambda.xyz/splices-toggles.html) to
add the value of variables to templates and [standard control
structures](https://maud.lambda.xyz/control-structures.html) to handle Option
values or whatever.

I've wanted to move my site over to use them generically (and I've actually done
most of the work to use them already for a lot of the shortcodes like the
conversation snippets), so I took the time to do that. One of the easy to notice
differences is the HTML of any page on the site. Before there was a lot of
newlines and how everything is crammed into as few lines as possible.

Shockingly, doing all of these changes seems to have had _no noticeable_
difference in how browsers render my website. I'm surprised too.

As of right now the only things left using ructe are my RSS/Atom feeds and the
first implementation of conversation snippets. I am honestly afraid to change
that last one, so I'm leaving it as-is because it works and I don't want to
touch it.

Maybe this will make things load faster? I don't know. I'm not an HTML-ologist.

## Pictures on the Patrons page

My [patrons page](https://xeiaso.net/patrons) has traditionally had a list of
names. I thought I'd take advantage of rewriting the templates to make it a bit
nicer looking, so I grabbed the avatar field out of everyone's entries and told
my HTML template to put it in a fancy grid. This probably doesn't look nice on
phones, but it looks great on my desktop and iPad so it's probably good enough.

## Everything builds using flakes

I've maintained two builds of my site for a while. The Nix flakes one that I use
in testing and the legacy Nix build that I use because I still haven't converted
all of my projects on the server that runs this website to use flakes yet.

My site uses [flake-compat](https://github.com/edolstra/flake-compat) to build
the flakes build with a non-flakes environment. This is intended to make things
easier, but I will still have CI build both the flakes and non-flakes build just
in case.

## Xesite finally uses Xeact properly

I've tried to use Xeact a few times on my blog. Nearly all of those attempts
have been failures. I think this time will be different. I've had a share on
Mastodon button on the bottom of every page for a few years. The old button used
to use the browser `prompt()` function to query for your Mastodon instance and
then it would pop a new tab to a precomposed toot that would mention me (so I
can track usage of it).

More people use this than you'd think. Even more since [all of the everything
with Twitter](https://xeiaso.net/blog/rip-twitter) started to happen. While I
was rejiggering all of the backend stuff with templates, I thought I'd mix in
Xeact more properly and use [Xeact's JSX
support](https://xeiaso.net/blog/xeact-jsx) to automagically compile things.

I ended up with the "Share on Mastodon" dropdown you'll see at the footer of
this article. It's implemented using [this TypeScript
file](https://github.com/Xe/site/blob/main/src/frontend/mastodon_share_button.tsx).
I'm happy with this so far, it does everything the old one did but just a bit
better. It does use some invisible HTML elements to pass variables from the HTML
template to the Xeact component, which I would love to change to JSON in a
`<script>` tag or something, but this works for now.

---

Anyways, that's what I've been up to with Xesite! Thank you all for reading
this, I never thought I'd get to the point where I've been working on the same
project for almost 8 years. I usually pick up and abandon projects really
quickly, so it's very odd that I have a bigger one that sticks around like this.

I'm super thankful for all of you reading and promoting this blog around. I've
been using this blog to hone my writing talents and it's been really encouraging
to see things hit the way I've wanted them to.

You know, I wonder how many articles I've written:

```
$ ls ./blog ./talks | wc -l
324
```

Holy crap. I write a lot. Given that my first commit was on [January 31,
2015](https://github.com/Xe/christine.website/commit/a7092d46b4a6612fd77a85969c407043c665ed32),
that means that I've had this blog up for about 2855 days. Given I have 324
posts, this means that on average I have written an article about once every _9
days_. This is kind of astounding.

```
$ nix run nixpkgs#lua5_3
Lua 5.3.6  Copyright (C) 1994-2020 Lua.org, PUC-Rio
> 2855/324
8.8117283950617
```

Here's to many more posts! Let's see if we can get that number down!
