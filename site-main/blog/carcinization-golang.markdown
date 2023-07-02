---
title: "The carcinization of Go programs"
date: 2022-11-22
author: Heartmender
tags:
 - cursed
 - wasm
 - go
 - rust
---

<xeblog-hero ai="Waifu Diffusion v1.3 (float16)" file="crab-invasion" prompt="crabs, invasion, beach, palm trees, green hill zone, studio ghibli, xenoblade chronicles 2, pokemon, ken sugimori, thick outlines, ink"></xeblog-hero>

Sometimes you just need to embed this one library written in another language
into your program. This is a common thread amongst programmers time immemorial.
This has always been a process fraught with peril, fear, torment, and
lemon-scented moist towelettes for some reason.

Normally if you want to call a Rust function from Go, you have to go through
some middleman like [cgo](https://pkg.go.dev/cmd/cgo). This works and is
somewhat elegant for how utterly terrible a hack cgo is.

However, the main problem is that when you use cgo to link a Rust function into
a Go program, you need to copy around the shared object that Rust generates. You
can't check this shared object into your source tree (it needs to be unique per
OS distribution per OS per CPU architecture, much like normal dynamically linked
binaries are). It does work, but overall the developer experience is poor. Your
build is no longer one simple `go build`. Now you have to remember to run `cargo
build --release` and ensure that the resulting `.so`, `.dll`, or `.dylib` is in
the right path for the OS' dynamic linker to read from. It's a mess.

<xeblog-conv name="Mara" mood="hacker">This is such a big problem that at a
generic level that this is why [Nix and NixOS](https://nixos.org) exist. Imagine
how complicated this is when you get general-purpose OS components into the mix.
It's astounding that anything works at all.</xeblog-conv>

So what if I told you there was a way that we could ship _one_ binary from Rust,
have that work on _every_ platform Go supports, and not have to modify the build
process beyond a simple `go build`? Imagine how much easier that would be. It's
easy to imagine that such a thing would let users _not even know that Rust was
involved at all_, even if they consume a package or program that uses it.

I've done this with a package I call
[mastosan](https://github.com/Xe/x/tree/master/web/mastosan) and here's why it
exists as well as how I made it.

## Why

Mastodon stores toots in HTML and presents that HTML to API consumers. HTML is
very nice for a browser to display, but this is not as useful for a bot.
Especially if your goal is to send toots to a Slack webhook.

When you look at a toot such as this in the API:

<xeblog-toot url="https://pony.social/@cadey/109388957068451434"></xeblog-toot>

Its content looks something like this:

```html
<p>test mention <span class="h-card"><a href="https://vt.social/@xe" class="u-url mention">@<span>xe</span></a></span> so I can see what HTML mastodon makes</p>
```

Ideally we'd like it to look semantically identical in Slack, maybe something
like this:

```
test mention <https://vt.social/@xe|@xe> so I can see what HTML mastodon makes
```

This will display the link in Slack like any other hyperlink. As things get more
elaborate, Mastodon will do more semantic weirdness like invisible spans and
other things that make displaying things on Slack annoying. Imagine the
difference between these two things:

```
https:// tailscale.com/blog/introducing -tailscale-funnel/

https://tailscale.com/blog/introducing-tailscale-funnel/
```

One is much more easy to understand for humans than the other.

## How

One of the core features of the UNIX philosophy is the idea that programs are
simple filters that do _one thing well_ and then allow you to compose them into
new and interesting ways. If you've ever used `curl` and `jq` together to do
things like read data from a JSONFeed, you know how this is in practice:

```console
$ curl https://xeiaso.net/blog.json -qsSL | jq .items[0].title -r
The birdsong persists
```

I made a little program in Rust that uses
[lol_html](https://docs.rs/lol_html/latest/lol_html/) to take incoming
Mastodon-flavored HTML and emit slack-flavored markdown. Usage is simple:

```
$ echo '<p>test mention <span class="h-card"><a href="https://vt.social/@xe" class="u-url mention">@<span>xe</span></a></span> so I can see what HTML mastodon makes</p>' | ./testdata/mastosan.wasm
test mention <https://vt.social/@xe|@xe> so I can see what HTML mastodon makes
```

That's it. It takes input on standard input and returns the result on standard
output. This doesn't cleanly map to the WebAssembly flow, except if you use
[WASI](https://wasi.dev/) to bridge the gap. WASI gives WebAssembly programs
enough of a POSIX-like environment that most basic things can work, but here we
are only really using two major parts of it: standard input and standard output.

In Go, if you were running this as a normal OS subprocess, you'd probably write
some code like this:

```go
package foo

import (
    "bytes"
    "os/exec"
    "strings"
)

func HTML2Slackdown(input string) (string, error) {
    loc, err := exec.LookPath("mastosan")
    if err != nil {
        return "", err
    }
    
    fout := &bytes.Buffer{}
    cmd := exec.Command(loc)
    cmd.Stdin = bytes.NewBufferString(input)
    cmd.Stdout = fout
    if err := cmd.Run(); err != nil {
        return "", err
    }
    
    return strings.TrimSpace(fout.String()), nil
}
```

However this still depends on the program being compiled for your native OS and
distribution as well as present in a folder in your `$PATH`. This works, but
this is not ideal in the slightest.

Rust lets you build a binary that targets WASI with this compiler flag:

```
cargo build --target wasm32-wasi --release --bin mastosan
```

This will emit a several megabyte binary file in
`./target/wasm32-wasi/release/mastosan.wasm`. When you run it, it will do what
you want.

Now you need to use it from Go. There's many choices for this, but I chose to
use [wazero](https://wazero.io/). The overall flow of using this is similar to
using a subprocess with `os/exec`, but slightly different because we're
embedding WebAssembly. It will look like this:

```go
//go:embed testdata/mastosan.wasm
var mastosanWasm []byte

func HTML2Slackdown(ctx context.Context, text string) (string, error) {
    // create wazero runtime
	r := wazero.NewRuntime(ctx)
	defer r.Close(ctx)
    
    // load wasi environment into runtime
	wasi_snapshot_preview1.MustInstantiate(ctx, r)

    // set up standard output and standard input
	fout := &bytes.Buffer{}
	fin := bytes.NewBufferString(text)

    // create runtime configuration
	config := wazero.NewModuleConfig().WithStdout(fout).WithStdin(fin).WithArgs("mastosan")

    // compile the WASM module
	code, err := r.CompileModule(ctx, mastosanWasm)
	if err != nil {
		log.Panicln(err)
	}

    // run the WASM module
	if _, err = r.InstantiateModule(ctx, code, config); err != nil {
		return "", err
	}

	return strings.TrimSpace(fout.String()), nil
}
```

This is mostly the same thing. You set up the environment, load the WASM module
and then run it. The main difference is that instead of loading the binary as
machine code from the disk, I use
[go:embed](https://pkg.go.dev/embed#hdr-Strings_and_Bytes) to embed the
precompiled WebAssembly module into the binary. This means that the resulting Go
program will Just Work as long as the WebAssembly module is present in the place
it expects.

## Moar faster

One main drawback to this implementation is that it's a bit slow. It has to
compile the WebAssembly module _every time_ the function is called.

The wazero runtime and compiled WebAssembly module code can be lifted into a
package-level variable, like with [this
patch](https://github.com/Xe/x/commit/b61b59318be6544632ac1f64b1237bb17b2e7a32).
The main advantage this gives you is _speed_. After this patch, the WebAssembly
module is only ever compiled _once_, on application boot. Before this patch,
each invocation took about 0.2 seconds per run. Here's the benchmark results
after this patch:

```
BenchmarkHTML2Slackdown             1221            938774 ns/op
BenchmarkHTML2Slackdown-2           2293            488032 ns/op
BenchmarkHTML2Slackdown-6           3555            305505 ns/op
BenchmarkHTML2Slackdown-12          3897            297974 ns/op
```

It's gone down from 0.2 seconds in the best case to _0.3 milliseconds_ in the
best case. This is at least a 1000x increase in performance, with most of the
time probably being spent in the HTML parser rather than being spent in anything
else.

I think this is going to more than meet my needs both personally and at work.
I'm going to have to try this a bit more against random Mastodon messages to see
if it does what I want. It's cool to be able to merge two incompatible worlds
together and I'm excited to see what I can do in the future with this.

<xeblog-conv name="Cadey" mood="coffee">Darn you shitposting coworkers nerd
sniping the poor DevRel on their well-earned week off!</xeblog-conv>
