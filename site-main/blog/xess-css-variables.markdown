---
title: "Xess 2: CSS variable edition"
date: 2022-11-06
tags:
 - css
 - frontend
 - nix
 - noxp
---

<xeblog-hero ai="Waifu Diffusion v1.3 (float16)" file="aoi-chan" prompt="1girl, fox ears, blue hair, blue eyes, paintbrush, canvas, easel, chibi, hoodie, smile, solo, very colorful, heart, pupils"></xeblog-hero>

As a hacker with too many side projects, I like to have a certain _look_ to my
websites that makes it instantly identifiable. I have a very brutalist approach
to web design that makes it _very easy_ to get things off the ground and get
hacking.

One of my longer-standing projects is a CSS framework called
[Xess](https://github.com/Xe/Xess). Xess is my go-to CSS file when I just need
to throw some words on a page. I've used it for at least the following projects:

- [When Then Zen](https://when-then-zen.christine.website)
- [waifud](https://github.com/Xe/waifud)'s admin panel
- [hlang](https://h.christine.website)
- [Printer Facts](https://printerfacts.cetacean.club/)

And other internal projects at jobs that I can't talk about due to NDA
restrictions. The really big thing that it does is lets you use _normal semantic
HTML_ and then just tries to make _that_ look pretty for you. It's a classless
framework (you don't need to use CSS classes to make it work) and I love it so
much.

However, after using it for a while, it's started to get very bland and
repetitive. Everything looks _the same_. This is getting boring to me. I've been
considering various ways to fix this, but I recently had a golden moment of
inspiration when I saw one of my favorite Fediverse bots come across my feed:

<xeblog-toot url="https://botsin.space/@randomColorContrasts/109212579424662451"></xeblog-toot>

I've been following
[@randomColorContrasts@botsin.space](https://botsin.space/@randomColorContrasts)
for years on the Fediverse. Twice a day it generates two colors with good
contrast and posts an example image with them in it. This gets you results that
look like this:

<div style="padding=1rem;color:#F1DD15;background:#0E102E;"><h2>Example
thing</h2><p>I'm baby umami truffaut beard hashtag squid mixtape tilde photo
booth etsy drinking vinegar humblebrag intelligentsia. Squid shabby chic
pinterest yuccie. Lomo organic pork belly man bun chillwave. Mlkshk coloring
book chia, kinfolk shoreditch pabst edison bulb marfa salvia vibecession fit
tumblr stumptown heirloom mixtape. Ugh yes plz shabby chic ennui pinterest
drinking vinegar tbh truffaut. Church-key big mood distillery trust fund
asymmetrical cray cliche. Tonx typewriter poutine before they sold out try-hard
umami fashion axe post-ironic JOMO normcore gochujang man bun glossier
butcher.</p></div>

This looks pretty great as-is, but Xess has more than just text being styled.
Xess also styles links, blockquotes, and code blocks. I really wanted those
colors to be derived on the fly and then I stumbled across [HSL calculations on
the fly with CSS
variables](https://elad.medium.com/why-css-hsl-colors-are-better-83b1e0b6eead).
This piqued my interest. I could pick three basic hues and use those to
dynamically generated everything else. I did some hacking and now I am happy to
announce Xess 2.0.

## Xess 2.0

Here are some screenshots of one of the themes I created for this: cherry

<xeblog-toot url="https://pony.social/@cadey/109298544735230581"></xeblog-toot>

My favorite part about all of this is how easy it is to customize a Xess theme.
You only need to change _three_ variables to recolor the page: one for the
background color, one for the text color, and one for the "accent" color (used
for selection, blockquotes and links).

Don't believe me? Here's the [theme
file](https://github.com/Xe/Xess/blob/master/custom/cherry.css) for the cherry
I showed off earlier:

```css
:root {
    --background-color: 0;
    --text-color: 43;
    --accent-color: 344;

    --width: 80ch;
    --padding: 0;
}
```

That's it. Just three colors, the max-width of the page and how much padding you
want around the main element. From here these can be infinitely customized to
your heart's content.

Here are some other themes I've cooked up:

<xeblog-toot url="https://pony.social/@cadey/109298533570134274"></xeblog-toot>

<xeblog-toot url="https://pony.social/@cadey/109298538717562125"></xeblog-toot>

I could easily see this being used with some kind of dynamic generation of theme
colors based on user settings or even dynamically generated at a per-user level.

## Customization with Nix flakes

I have a slight reputation as being a NixOS user. One of the biggest things that
I like to do with Xess is consume it from [Nix
flakes](https://nixos.wiki/wiki/Flakes) so that the CSS will automatically be
squashed into one tiny file. I have made a Nix function that generates a package
with the CSS customizations into one big file so that I can solve two problems
at once:

* Making per-project customizations of Xess without having to build things
  manually or edit Xess itself
* Make that a Nix function so that I don't have to think about it too much at
  the build stage

If you want to use a customized version of Xess in your Nix build, here's what
you need to do:

First, import Xess into your flake and add its outputs to your flake outputs:

```nix
{
  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
    utils.url = "github:numtide/flake-utils";
    xess.url = "github:Xe/Xess";
  };
  
  outputs = { self, nixpkgs, utils, xess }@inputs:
    utils.lib.eachDefaultSystem (system:
      let pkgs = nixpkgs.legacyPackages.${system};
      in {
        # ...
      });
}
```

Make your theme somewhere in your repo. I'm going to assume it's at
`./css/theme.css`. Then you can build a custom version of Xess with this:

```nix
in {
  packages = {
    xess = Xess.packages.${system}.customized ./css/theme.css;
  };
  
  # ...
}
```

You can test this with `nix build`:

```shell
$ nix build .#xess
```

And then view it in your browser with `python -m http.server`:

```shell
$ python -m http.server
```

Add `/result/sample.html` to the end of the URL that python generates for you,
and then you can view your CSS changes in all their glory. The CSS file that
Xess generates will be in `$out/static/css/xess.css`. You can use this with
something like `pkgs.symlinkJoin` to make sure things percolate out to the right
place:

```nix
in {
  packages = rec {
    xess = Xess.packages.${system}.customized ./css/theme.css;
    bin = doSomeBuild;
    
    # the default output
    default = pkgs.symlinkJoin {
      name = "myProject-${bin.version}";
      paths = [ bin xess ];
    };
  };
  
  # ...
}
```

And then as long as your website expects to pull things from
`/static/css/xess.css`, everything will just work! Nix will take your
customizations, splice them into Xess and then emit a composite CSS file with
everything in it.

<xeblog-conv name="Mara" mood="happy">Also, if you're an existing user of Xess
via Nix, this will _not_ break your builds. When you use the default package in
the Xess flake, it will build the "classic" version of Xess without any of the
HSL customizations.</xeblog-conv>

That's it! I'm really glad that I'm bringing Xess into the future and making it
more extensible for the next few years of effort. I'm excited to hear what you
can do with Xess. Be sure to let me know what fun you get up to on
[Mastodon](https://pony.social/@cadey)!
