---
title: "[talk] The carcinization of Go programs"
date: 2023-03-24
slides_link: https://drive.google.com/file/d/1ANxRJPzNeKbLogZz0wCH_o9E9jNLypja/view?usp=sharing
tags:
 - wasm
 - wazero
 - rust
 - golang
skip_ads: true
---

<xeblog-talk-warning></xeblog-talk-warning>

<xeblog-video path="talks/2023/wazero-lightning"></xeblog-video>

## Transcript

<xeblog-conv standalone name="Cadey" mood="enby">This is a lightning
talk version of [this
post](https://xeiaso.net/blog/carcinization-golang).</xeblog-conv>

<xeblog-slide name="2023/wazero-lightning/01" essential></xeblog-slide>

Computers are complicated. Programming computers is even more
complicated. Sometimes you have a square hole, but the square peg you
need is written in Rust and the rest of your program is written in Go.
Linking these two worlds together like this is a painful process
fraught with peril, fear, terror, utter torment, and lemon-scented
moist towelettes.

<xeblog-slide name="2023/wazero-lightning/02"></xeblog-slide>

Normally if you want to call a Rust function from Go, you need to
expose a C interface in Rust, and then bind to that C interface with
cgo. This does work. It's a thing you can do, but it only works the
most reliably at very small scales. The main problem with doing this
is that it breaks people's semantic expectations of how tools should
work.

<xeblog-slide name="2023/wazero-lightning/03"></xeblog-slide>

When Rust compiles `.so` files that link against your program, they
also link against system libraries. This isn't normally a problem,
except if you have people like me around that use a hipster distro
where everything is different. You also have to maintain a separate
dot s o file for each OS and CPU architecture combination you support.
Your build step is no longer `go build`, it's in Makefile land.

<xeblog-slide name="2023/wazero-lightning/04"></xeblog-slide>

So, imagine a world where you could just put one binary blob into your
git repo, and have that Just Work everywhere. Without making your
build more complicated than `go build`. Without configuration when
people are building the code. Imagine how much easier that would be.

<xeblog-slide name="2023/wazero-lightning/05"></xeblog-slide>

I bet you can guess where I'm going with this, I'm talking about the
carcinization of Go programs via WebAssembly. This is how I snuck Rust
into a Go shop.

<xeblog-conv name="Mara" mood="hacker">The "carcinization" refers to
the evolutionary tendency of programs becoming either crabs or trees
when time stretches to infinity. If you can imagine library use as
evolution, then this joke makes more sense.</xeblog-conv>

<xeblog-slide name="2023/wazero-lightning/06"></xeblog-slide>

Why?

<xeblog-slide name="2023/wazero-lightning/07"></xeblog-slide>

Mastodon. Mastodon uses a protocol called ActivityPub to federate
posts. ActivityPub posts are formatted in HTML. Reading Mastodon posts
from the API returns HTML. We want to show these posts off in a Slack
channel, and Slack doesn't support using your own HTML. It has its own
bespoke markup format called Slackdown because of course it does.

<xeblog-slide name="2023/wazero-lightning/08"></xeblog-slide>

There was nothing off of the shelf to handle this in Go. I assume that
this problem is fairly novel. Anything that was close to this just
made atrocities out of the text in ways that I couldn't customize.

<xeblog-conv name="Aoi" mood="wut">I guess some solutions could be out
there, but just locked in closed-source repos.</xeblog-conv>

<xeblog-slide name="2023/wazero-lightning/09"></xeblog-slide>

Then I remembered a Rust library that I use on my blog:
[lol_html](https://docs.rs/lol_html/latest/lol_html/). lol_html is a
streaming HTML parser/rewriter. I use it on my blog for all my HTML
shortcodes and I know it lets you mangle HTML into other formats with
element handlers. Mastodon has a list of HTML tags it will send out
over the API, so I used that to assemble a little test program in
Rust.

<xeblog-slide name="2023/wazero-lightning/10"></xeblog-slide>

When I wrote this program, I made it in a fairly naiive way. I took
HTML over standard input and had it spit out slackdown on standard
output. 

This idea just so happens to be one of the core parts of the Unix
philosophy: programs should be filters that take input in one form and
then transform it to output in another form. This made it trivial to
test against arbitrary HTML from the Mastodon API and get things down
to the output format I wanted.

<xeblog-slide name="2023/wazero-lightning/11"></xeblog-slide>

Then comes integration. I already knew I didn't want to deal with the
surreal horror that is dynamically linking Rust code into Go (but I
could figure it out if I needed to), so on a lark I decided to just
try compiling it to WebAssembly with [WASI](https://wasi.dev) and see
if that works.

<xeblog-slide name="2023/wazero-lightning/12"></xeblog-slide>

It did. I then applied some optimizations and got it down to a 200
kilobyte ball of mud that did exactly what I needed. Then all I needed
to do was glue it into the Go program with Wazero.

<xeblog-slide name="2023/wazero-lightning/13"></xeblog-slide>

Wazero made this trivial. I overrode the input and output buffers,
then had it kick things off as needed. At first my code was very lazy
and slow, but it worked. The only build step was `go build`, just like
I wanted. I committed the WASM blob to git and shipped the hack. It
worked perfectly.

<xeblog-slide name="2023/wazero-lightning/14"></xeblog-slide>

The last part was making it faster. With some guidance from #wazero on
Slack, I managed to get each call of this function down to two hundred
microseconds. That's faster than the equivalent code in Go using its
[HTML parsing library](https://pkg.go.dev/golang.org/x/net/html) that
I only found out existed about a week ago.

<xeblog-slide name="2023/wazero-lightning/15"></xeblog-slide>

And now this atrocity is shipped to production and holds together the
Mastodon post announcement service. Most people aren't aware that it's
a thing, and it runs fast enough that nobody really cares that it's a
programming turducken of Go, Rust and WebAssembly. It works perfectly 
and it wouldn't be possible without the efforts of the Wazero team.

<xeblog-slide name="2023/wazero-lightning/16" essential></xeblog-slide>

I'm Xe Iaso, and I hoped you enjoyed my talk about programming crimes.

I do developer relations at Tailscale as the Archmage of
Infrastructure. If you have any questions, please don't hesitate to
reach out at [wazero2023@xeserv.us](mailto:wazero2023@xeserv.us). I'm
more than happy to answer them.

Congrats to the Wazero team for the 1.0 release! Be well, all!

## With apologies to

- Renee French
- The Rust community
- The Wazero team
- Hbomberguy
