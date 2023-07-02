---
title: "Site Update: New Fonts"
date: 2023-03-19
series: site-update
tags:
 - font
 - design
 - CSS
 - woff2
---

Hey all! You may have noticed things looking a little different on the
site. I have finally decided to adopt a custom font family for my
blog. Unless your browser overrides the fonts that websites use, you
are likely looking at the first attempt at the Iosevka Iaso font
family. This is a custom version of the
[Iosevka](https://typeof.net/Iosevka/) font that I designed for my
online brand image. It includes three fonts that serve different
purposes:

* <span style="font-family: Iosevka Aile Iaso;font-size:1.25rem">Iosevka Aile Iaso -
  screen prose, sans serif</span>
* <span style="font-family: Iosevka Curly Iaso;font-size:1.25rem">Iosevka Curly Iaso -
  code and terminals</span>
* <span style="font-family: Iosevka Etoile Iaso;font-size:1.25rem">Iosevka Etoile Iaso -
  screen titles, printed text, has serifs</span>

I'm happy with the results so far and I am going to continue to do
research and design work to make this better. In the future I plan to
extend the set with a variant optimized for dialogue.

<xeblog-hero ai="Anything" file="park-drawing" prompt="1girl, light green hair, green eyes, black hoodie, long hair, outdoors, seattle, picnic table, wearing headphones, sketchpad, drawing, pondering, ponytail, space needle"></xeblog-hero>

<xeblog-conv name="Cadey" mood="enby">I also made the base font on the
webpage a bit bigger. Let me know if this is too big!</xeblog-conv>

## Why?

<xeblog-conv name="Mara" mood="hmm" standalone>I don't get why you'd
need your own font though. Why aren't system fonts good
enough?</xeblog-conv>

In general, my website used to look inconsistent across different
platforms. I had optimized things for macs with the
[Menlo](https://en.wikipedia.org/wiki/Menlo_(typeface)) typeface as
the default view of my website. This has worked for a long time, and
most of my readers use macs anyways. However, I've been wanting to do
a design refresh on this blog for a while and haven't really found a
good stepping stone into it.

<span style="font-family:sans-serif;font-size:1.125rem">Plus, it looks
really weird to use a non-fixed-width font on this website. It looks
really out of place and jarring. The fact that so much of my brand
design is centered around monospace fonts really makes it hard to
change anything without making everything look <em
style="font-size:1.125rem">annoyingly different</em>. The fact that
the CSS I use assumes the use of a monospace font certainly doesn't
help either.</span>

Monospace fonts are also [very bad for
prose](https://ux.stackexchange.com/questions/48753/does-the-use-of-monospaced-fonts-negatively-affect-legibility-of-article-text).
My blog has lots of prose. This is a problem. I've had dyslexic
friends of mine lament that my blog's monospace font makes it harder
to identify the shapes of words, and some have gone as far as using
their browser to override everything to their chosen easy to read
font. This probably needs to be fixed.

At the same time, if I make something like this, I want to be able to
create an opinionated set of fonts that I can easily plug into other
projects besides this blog. This would allow me to have things look
consistent across browsers, projects, and even print media. I've never
really had a "brand design" per se, and I want to experiment with this
to see if I can get something that I like.

I think I have created a large part of that with the Iosevka Iaso font
family. The Iosevka Aile Iaso font is quasi-proportional, meaning that
it looks something halfway between a proportional font like I'd like
to have and the monospace font that I used to have. This should
hopefully make the transition less jarring.

## Implementation details

The core of the implementation of this font is thanks to the Iosevka
font being [customizable](https://typeof.net/Iosevka/customizer). If
you've never done it before, take a moment to play around with the
customizer. It's nuts. You have so much control over what the font
does and how every letter is rendered. Want ligatures? You can pick
the set you want. Want the `T` to look different? You can do that.

These build settings are even exposed to Nix, so you can make
[something like my `iosevka`
flake](https://github.com/Xe/iosevka/blob/main/flake.nix) and then
have that get cached around so other projects can pull it in as a
flake input. This allows me to pull in the new font changes and feel
confident that things are consistent. This ends up making the
dependency graph look something like this:

![](/static/img/iaso-fonts-depgraph.svg)

Everything pulls from the `iosevka` flake, so making a change there
will percolate out to everything else.

One of the difficulties I ran into at first was the fact that Iosevka
[supports so many characters](https://typeof.net/Iosevka/specimen).
Each one of the `.ttf` files in the Iaso family is over a megabyte. At
some level, this is really great for all of my other production work.
I can use this on slides and in my editor while remaining fairly
confident that I'll be able to see everything I need to.

This is not so great for websites though. Even with `.woff2`
compression of the fonts, it still added up to something like 500 KB
per font family. With up to three fonts per page, this means that I
would be wasting about 1.5 megabytes of egress data per user with
characters that my blog will never have.

Thanks to some meddling courtesy of [iliana](https://iliana.fyi), I
found a command that lets me squish the fonts down to the bare
minimum: `pyftsubset`. I use it to remove most of the characters in
the web-optimized fonts with this command:

```shell
pyftsubset \
    $ttf \
    --output-file="$name".woff2 \
    --flavor=woff2 \
    --layout-features=* \
    --no-hinting \
    --desubroutinize \
    --unicodes="U+0000-00A0,U+00A2-00A9,U+00AC-00AE,U+00B0-00B7,\
      U+00B9-00BA,U+00BC-00BE,U+00D7,U+00F7,U+2000-206F,U+2074,\
      U+20AC,U+2122,U+2190-21BB,U+2212,U+2215,U+F8FF,U+FEFF,\
      U+FFFD"
```

This giant block of text takes a input `.ttf` file generated by Nix
and then removes most of the characters in it except the ones needed
for programming, English, Extended Latin, French, German, Italian, and
I believe Swedish. This should cover all of the letters I need. This
command squashes down the `.woff2` files from 500 KB to about 12 KB
and change, making my fonts about as large as the conversation snippet
stickers.

<xeblog-conv name="Aoi" mood="cheer">Wow! That's small!</xeblog-conv>

I think that this will allow me to get the benefits of a consistent
design across my websites while I work on a more comprehensive
rethinking of my blog's and Xess' design to allow for an easy to
maintain, integrate, and reason about font experience.

## Usage

If you want to use this font family for some reason, add this to your
HTML `<head>` section:

```html
<link rel="stylesheet" href="https://cdn.xeiaso.net/static/css/iosevka/family.css" />
```

Or this to your CSS:

```css
@import url(https://cdn.xeiaso.net/static/css/iosevka/family.css);
```

From here you should attach the fonts to their "default roles" such as
by doing this:

```css
body {
  font-family: "Iosevka Aile Iaso", sans-serif;
  font-size: 16px;
}

code, pre {
  font-family: "Iosevka Curly Iaso", monospace;
  font-size: 14px;
}

h1, h2, h3, h4, h5, h6 {
  font-family: "Iosevka Etoile Iaso", serif;
}
```

I'd suggest setting the base font size to be 16px, as that seems to be
the best balance of text density and readability. Code can be a smidge
smaller, but be sure to balance things so that it's not uncomfortable
to read. Comfort is key.

## TODO

These fonts aren't perfect yet, here's my short list of things to
change when I figure out how:

* The fonts are very vertical and not horizontal. This is fine for
  many people, but this can be a readability issue for others.
* I would like to add my sigil to the font as a custom icon. I need to
  figure out how to do this.
* I need to do a more detailed study of how people with dyslexia can
  read this font. I have a very mild level of dyslexia, but I can read
  this just fine. More information is required.

Overall I'm quite happy with the results so far. It's nice to have a
custom font that I can call my own. I'm going to iterate on this like
the rest of my projects and hopefully I'll have something more
exciting to show off in the near future.

<xeblog-conv name="Cadey" mood="coffee">Holy crap, font terminology is
pretentious sounding as all hell. You have to worry about ligation,
investigate specimens, and all that jazz. It's a lot to take
in.</xeblog-conv>
