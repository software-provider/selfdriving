---
title: How to use a fork of the Go compiler with Nix
date: 2023-03-28
tags:
 - golang
 - nix
---

<xeblog-hero ai="Fluff Proto-r10" file="coffee-gopher" prompt="rodent, gopher, blue fur, blue hair, blue skin, calarts, solo, male, laptop, coffee shop, detailed background, anthro, coffee mug, happy, black nose, best quality, highly detailed, eyes closed"></xeblog-hero>

Sometimes God is dead and you need to build something with a different
version of Go than [upstream released](https://go.dev/dl). Juggling
multiple Go toolchains is _possible_, but it's not very elegant.

However, we're in Nix land. We can do anything\*.

<xeblog-conv name="Aoi" mood="coffee">\*with sufficent hackery.</xeblog-conv>

I got accepted to [Gophercon EU](https://gophercon.eu/speakers) and a
lot of it involves doing weird things with WebAssembly and messing
with assumptions people make about how filesystems work. Given that
most of my audience is going to be Go programmers and that I'm already
going to be cognitively complicating how core assumptions about
filesystems work, I want to show my code examples in Go when at all
possible.

Go doesn't currently support [WASI](https://wasi.dev), but there is a
[CL in progress](https://go-review.googlesource.com/c/go/+/479627)
that adds the port under the name `GOARCH=wasm GOOS=wasip1`. I wanted
to pull this into my monorepo's Nix flake so that I can run `gowasi
build foo.go` and get `foo.wasm` in the same folder to experiment with.

<xeblog-conv name="Mara" mood="hacker">A CL in the Go ecosystem is a
change list or change log. You can think about it as analogous to a
pull request in GitHub.</xeblog-conv>

Turns out this is really easy. In order to do this, you need to do
three things:

- Add a flake input for the fork of Go in question
- Create a build of Go with that fork and a fabricated VERSION file
- Create the wrapper script and populate it in your devShell

## Add a flake input

Nix flake inputs don't let you just import other Nix flakes; they also
can be used for any git repository, such as the [Go source
tree](https://github.com/golang/go). To create an input with an
arbitrary fork of Go (such as [Xe/go](https://github.com/Xe/go)), do
this:

```nix
# go + wasip1
wasigo = {
  url = "github:Xe/go/wasip1-wasm";
  flake = false;
};
```

The important part is `flake = false`, that tells Nix to treat it as a
raw repository and not assume that it's a Nix flake. The `wasigo`
variable can be used as the path to the extracted tarball and will
contain the following attributes:

- `lastModified` - The last modified time for the git repo in unix
  time
- `lastModifiedDate` - The last modified time in datetime format
- `narHash` - The Nix ARchive hash in base64-sha256 form
- `outPath` - The Nix store path associated with this flake input
- `rev` - The full git hash for this flake input
- `shortRev` - The short form of the git hash

Add it as an argument to your `outputs` function.

## Build a custom toolchain

It's common to declare a bunch of variables that your flake uses
immediately inside your `outputs` function [like
this](https://github.com/Xe/x/blob/95c888fc35692228d7951f461508cd59fae0f691/flake.nix#L50),
so I'm going to assume that you are doing this. Add a variable for
your Go fork (eg: `wasigo'`):

```nix
wasigo' = pkgs.go_1_20.overrideAttrs (old: {
  src = pkgs.runCommand "gowasi-version-hack" { } ''
    mkdir -p $out
    echo "go-wasip1-dev-${wasigo.shortRev}" > $out/VERSION
    cp -vrf ${wasigo}/* $out
  '';
});
```

<xeblog-conv name="Aoi" mood="wut">Why are you using a `'` and calling
it `wasigo-prime`?</xeblog-conv>
<xeblog-conv name="Cadey" mood="enby">If I don't name it something
else, I will create an infinitely recursive definition. Nix is lazy
and only evaluates things when it needs to. Making a binding called
`wasigo` and using the name `wasigo` inside that will create infinite
recursion when it is evaluated. I don't know of a better name for
this, but a common pattern in Nix land is to use primes (`'`) for
distinct values with the same name. Just like in
Haskell.</xeblog-conv>
<xeblog-conv name="Aoi" mood="wut">What about that `VERSION`
file, what's that there for?</xeblog-conv>
<xeblog-conv name="Cadey" mood="enby">That is there to tell the Go
compiler toolchain what version it is. When you clone a git repository
into the Nix store, all of the git metadata is purged from the
checkout (because it's not byte-for-byte reproducible and random
changes there could cause unwanted rebuilds of a lot of packages). If
the `VERSION` file doesn't exist, the Go toolchain will try to
discover what version it is from the `git` metadata, which doesn't
exist. This file lies to the toolchain so that builds
work.</xeblog-conv>
<xeblog-conv name="Aoi" mood="cheer">I see, thanks!</xeblog-conv>

## Make a wrapper script

In many cases, you can just add `wasigo'` to your `devShell`
`buildInputs` and you'll be fine. In this case, we want to have a
separate command that pre-configures the `GOOS` and `GOARCH`
environment variables to target WASI. The
[`pkgs.writeScriptBin`](https://ryantm.github.io/nixpkgs/builders/trivial-builders/#trivial-builder-writeText)
trivial builder lets you write an arbitrary string to the Nix store as
a binary. You can use this to create a wrapper script:

```nix
gowasi = pkgs.writeShellScriptBin "gowasi" ''
  export GOOS=wasip1
  export GOARCH=wasm
  exec ${wasigo'}/bin/go $*
'';
```

This will create a file named `bin/gowasi` in a Nix package that will
set the correct environment variables and then execute the version of
Go that was just compiled. It will look something like this:

```sh
#!/nix/store/0hx32wk55ml88jrb1qxwg5c5yazfm6gf-bash-5.2-p15/bin/bash
export GOOS=wasip1
export GOARCH=wasm
exec /nix/store/px67cnp39lzynhknqqjjn9c3b838qnw9-go-1.20.2/bin/go $*
```

<xeblog-conv name="Mimi" mood="happy">The exec builtin command in Bash
is used to execute a command that completely replaces the current
shell process. The original shell process is destroyed and overwritten
by the new command. Any commands after the exec command in the script
do not get executed.</xeblog-conv>

And then you can go off to the races and compile things to your
heart's content!

## Overriding buildGoModule for that version of Go

If you want to build go modules using this version of Go, you need to
make your own `buildGoModule` analog:

```nix
buildGoWasiModule = pkgs.callPackage "${nixpkgs}/pkgs/build-support/go/module.nix" {
  go = wasigo';
};
```

Then use `buildGoWasiModule` like you would `buildGoModule`.

To force it to build webassembly modules, you will need to override
the `GOOS` and `GOARCH` attributes in `wasigo'`:

```nix
wasigo' = {
  # ...
} // {
  GOOS = "wasip1";
  GOARCH = "wasm";
};
```

This will force the Go compiler to output WebAssembly binaries, but
they will be put in `$out/bin/wasmp1_wasm/name` without the `.wasm`
suffix. This may not be ideal in some cases, but this is a limitation
in how `GOEXE` is not correctly threaded through the `buildGoModule`
stack when it is hacked like this.
