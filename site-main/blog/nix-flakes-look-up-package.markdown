---
title: "How to look up a Nix package's Nix store path from flake inputs"
date: 2022-08-06
tags:
 - nix
 - nixos
 - flakes
series: nix-flakes
---

<xeblog-hero file="fall-archons" prompt="The fall of the Archons, colored pencil drawing, fireball spell, bright sky, digital art, lake of fire"></xeblog-hero>

Sometimes God is dead and you need to figure out what the version of a package
in your Nix flake's inputs is. With flakes, you can figure this out using `nix
eval` on a flake reference, but what the hecc is a flake reference?

Every URL has a "fragment", or the thing after the `#` that mainly tells the
browser where to scroll to. Nix flakes uses this to allow you to sub-index on
flakes by either name or URL. This allows you to address packages like tmux as:

```
nixpkgs#legacyPackages.x86_64-linux.tmux
```

Or my site's code as:

```
github:Xe/site#packages.x86_64-linux.default
```

Or even some [arbitrary git repo](https://tulpa.dev/cadey/printerfacts):

```
git+https://tulpa.dev/cadey/printerfacts.git?ref=main#defaultPackage.x86_64-linux
```

<xeblog-conv name="Mara" mood="hacker">The flake reference is what drills down
into the flake so that Nix can tell the difference between part of the path to
the repository and the path inside the flake. It usually works on the `outputs`
of a flake that you can find out with `nix flake show`:</xeblog-conv>

```
$ nix flake show github:Xe/site
github:Xe/site/5a50cacef85c9e3b9695a2749359b27d0530f86a
├───devShell
│   ├───aarch64-linux: development environment 'nix-shell'
│   └───x86_64-linux: development environment 'nix-shell'
├───nixosModules
│   ├───aarch64-linux: NixOS module
│   └───x86_64-linux: NixOS module
└───packages
    ├───aarch64-linux
    │   ├───bin: package 'xesite-2.4.0'
    │   ├───config: package 'xesite-config-2.4.0'
    │   ├───default: package 'xesite-2.4.0'
    │   ├───posts: package 'xesite-posts-2.4.0'
    │   └───static: package 'xesite-static-2.4.0'
    └───x86_64-linux
        ├───bin: package 'xesite-2.4.0'
        ├───config: package 'xesite-config-2.4.0'
        ├───default: package 'xesite-2.4.0'
        ├───posts: package 'xesite-posts-2.4.0'
        └───static: package 'xesite-static-2.4.0'
```

<xeblog-conv name="Mara" mood="happy">Tracing through things, you can see how
you can go from the package name `default` to get
`github.com:Xe/site#packages.x86_64-linux.default` as the full legal name of the
package!</xeblog-conv>

You can use the `nix eval --raw` command to figure out what the nix store path
of those packages either is or would be:

```
$ nix eval --raw nixpkgs#legacyPackages.x86_64-linux.tmux
/nix/store/3mn362yak70vq137wnryi7whgvq2cmn2-tmux-3.3a

$ nix eval --raw github:Xe/site#packages.x86_64-linux.default
/nix/store/v3lwgh6vk03yw3aq3j398g613ff3gja0-xesite-2.4.0

$ nix eval --raw git+https://tulpa.dev/cadey/printerfacts.git'?'ref=main#defaultPackage.x86_64-linux
/nix/store/7df21bqkrx2csazc2s8ji8chaabbpqhs-printerfacts-0.3.1
```

However, this operates on the latest version by default. It won't operate on the
current version in your Nix flake. If you need to find it out from the current
flake, pass the `--inputs-from` flag with the current directory `.`:

```
$ nix eval --inputs-from . --raw nixpkgs#legacyPackages.x86_64-linux.tmux
/nix/store/5sz57bwrlwhv1l8h4c4sq1bl1fhmcwfw-tmux-3.3a
```

This will use the `nixpkgs` defined in your current directory's `flake.lock`
file. You can then share that store path with people to share in your tale of
woe as to why a service is crashing when you try and update it.

<xeblog-conv name="Mara" mood="hacker">You can also pass in an arbitrary version
for a flake URL if you specify it in the URL parameters with `ref`. You can do
it like this: `github:Xe/site?ref=5a50cacef85c9e3b9695a2749359b27d0530f86a` or
by specifying the commit hash as an additional path parameter as in
`github:Xe/site/5a50cacef85c9e3b9695a2749359b27d0530f86a`. Here's an example for
[this xesite
commit](https://github.com/Xe/site/commit/5a50cacef85c9e3b9695a2749359b27d0530f86a):</xeblog-conv>

```
$ nix flake show github:Xe/site/5a50cacef85c9e3b9695a2749359b27d0530f86a
github:Xe/site/5a50cacef85c9e3b9695a2749359b27d0530f86a
├───devShell
│   ├───aarch64-linux: development environment 'nix-shell'
│   └───x86_64-linux: development environment 'nix-shell'
├───nixosModules
│   ├───aarch64-linux: NixOS module
│   └───x86_64-linux: NixOS module
└───packages
    ├───aarch64-linux
    │   ├───bin: package 'xesite-2.4.0'
    │   ├───config: package 'xesite-config-2.4.0'
    │   ├───default: package 'xesite-2.4.0'
    │   ├───posts: package 'xesite-posts-2.4.0'
    │   └───static: package 'xesite-static-2.4.0'
    └───x86_64-linux
        ├───bin: package 'xesite-2.4.0'
        ├───config: package 'xesite-config-2.4.0'
        ├───default: package 'xesite-2.4.0'
        ├───posts: package 'xesite-posts-2.4.0'
        └───static: package 'xesite-static-2.4.0'
```

<xeblog-conv name="Cadey" mood="coffee">I have reached a point where googling
for these questions gets me results on my own blog. Of course they didn't help
me in the moment. That's why this post exists. Maybe I'll be able to fix my
Matrix server returning that a database migration failed so that I can update my
homelab machines because that failing makes the whole deploy rollback. I really
hate synapse and I regret setting up my own Matrix homeserver.</xeblog-conv>

