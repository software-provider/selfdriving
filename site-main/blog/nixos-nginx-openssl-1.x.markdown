---
title: "How to make NixOS compile nginx with OpenSSL 1.x"
date: 2022-10-29
tags:
 - openssl
 - nginx
series: nixos
---

<xeblog-hero ai="Waifu Diffusion v1.3 (float16)" file="alrest-orcas" prompt="cloud sea, xenoblade chronicles 2, azurda, blue sky, giant tree, orca, 1girl, red hair, katana"></xeblog-conv>

One of the strengths of NixOS is that you can use NixOS modules to do things
like override versions of packages so that you can customize what software is
running on your computer. You can use this to manually patch programs, or
alternatively override dependencies with other versions. Today I'm going to show
you how to use an overlay to force NixOS to rebuild
[nginx](https://nginx.org/en/) with OpenSSL 1.1.1 instead of OpenSSL 3.x. You
may want to do this if you want to reduce risks involved with the [CRITICAL
security issue announced for OpenSSL
3.x](https://xeiaso.net/blog/openssl-3.x-secvuln-incoming) (OpenSSL 1.1.1 isn't
listed as CRITICAL).

Open your `configuration.nix` file and add this inside the module block:

```nix
nixpkgs.overlays = [
  (final: prev: {
    nginxStable = prev.nginxStable.override { openssl = prev.openssl_1_1; };
  })
];
```

<xeblog-conv name="Mara" mood="hacker">If you are using NixOS 22.05, use the
package `openssl` instead of `openssl_1_1`.</xeblog-conv>

This will create an [overlay](https://nixos.wiki/wiki/Overlays) that will
replace the nginx package with a version that has OpenSSL replaced with the
OpenSSL 1.x package.

<xeblog-conv name="Mara" mood="hacker">You need to use `nginxStable` here
instead of `nginx` because `services.nginx.package` defaults to `nginxStable`.
Alternatively you can use something like this to change the nginx package
directly: `services.nginx.package = (pkgs.nginxStable.override { openssl =
pkgs.openssl_1_1; });` This may be ideal depending on facts and
circumstances.</xeblog-conv>

It uses an [override](https://nixos.org/manual/nixpkgs/stable/#chap-overrides)
to change the version of OpenSSL that is passed into the package build. This
works because packages in `nixpkgs` are defined something like this:

```nix
{ stdenv, openssl, fetchurl }:

stdenv.mkDerivation {
  # whatever is needed to build the software
}
```

Each of the inputs in the top line are _arguments_ to the package (which is
modeled as a function). When you use `.override`, you are overriding the
arguments you pass to the package functions. This means that when you use that
overlay I pasted, you will be overriding the version of OpenSSL passed to the
nginx build process, which will make nginx depend on OpenSSL 1.x.

Depending on the software in question, you should be able to use this strategy
to patch any other public-facing programs. The only catch is that software will
need to be compatible with OpenSSL 1.x.

<xeblog-conv name="Cadey" mood="coffee">You may want to remove this as soon as
NixOS unstable advances to OpenSSL 3.0.7.</xeblog-conv>

<xeblog-conv name="Mara" mood="hacker">Thanks to ckie for reviewing this post
for correctness!</xeblog-conv>
