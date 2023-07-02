---
title: How to Store an SSH Key on a Yubikey
date: 2022-05-27
series: howto
tags:
 - yubikey
 - security
---

SSH keys suck. They are a file on the disk and you can easily move it to other
machines instead of storing them in hardware where they can't be exfiltrated.
Using a password to encrypt the private key is a viable option, but the UX for
that is hot garbage. It's allegedly the future, so surely we MUST have some way
to make this all better, right?

<xeblog-conv name="Numa" mood="delet">\>implying there is a way to make anything
security related better</xeblog-conv>

Luckily, there is actually something we can do for this! As of [OpenSSH
8.2](https://www.openssh.com/releasenotes.html#8.2) (Feburary 14, 2020) you are
able to store an SSH private key on a yubikey! Here's how to do it.

<xeblog-conv name="Mara" mood="hacker">This should work on other FIDO keys like
Google's Titan, but we don't have access to one over here and as such haven't
tested it. Your mileage may vary. We are told that it works with the Google
Titan key that is handed out to Go contributors.</xeblog-conv>

First install `yubikey-manager` (see
[here](https://www.yubico.com/support/download/yubikey-manager/) for more
information, or run `nix-shell -p yubikey-manager` to run it without installing
it on NixOS), plug in your yubikey and run `ykman list`:

```console
$ ykman list
YubiKey 5C NFC (5.4.3) [OTP+FIDO+CCID] Serial: 4206942069
```

If you haven't set a PIN for the yubikey yet, follow
[this](https://docs.yubico.com/software/yubikey/tools/ykman/FIDO_Commands.html#ykman-fido-access-change-pin-options)
to set a PIN of your choice. Once you do this, you can generate a new SSH key
with the following command:

```
ssh-keygen -t ed25519-sk -O resident
```

<xeblog-conv name="Mara" mood="hacker">If that fails, try `ecdsa-sk`
instead! Some hardware keys may not support storing the key on the key
itself.</xeblog-conv>

Then enter in a super secret password (such as the Tongues you received as a kid
when you were forced into learning the bible against your will) twice and then
add that key to your agent with `ssh-add -K`. Then you can list your keys with
`ssh-add -L`:

```console
$ ssh-add -L
sk-ssh-ed25519@openssh.com AAAAGnNrLXNzaC1lZDI1NTE5QG9wZW5zc2guY29tAAAAIKgGePSwpBuHUhrFCRLch9Usqi7L0fKtgTRnh6F/R+ruAAAABHNzaDo= cadey@shachi
```

Then you can copy this public key to GitHub or whatever and authenticate as
normal. The private key is stored on your yubikey directly and you can add it
with `ssh-add -K`. You can delete the ssh key stub at `~/.ssh/id_ed25519_sk` and
then your yubikey will be the only thing holding that key.
