---
title: How to move away from RSA for SSH keys
date: 2023-01-04
tags:
 - OpenSSH
 - RSA
 - ed25519
 - security
 - sre
 - NixOS
---

<xeblog-hero ai="Stable Diffusion v1.5" file="volcano-bliss" prompt="a rolling green landscape by makoto shinkai, breath of the wild, active volcano, windows xp bliss, manga style, ((thick outlines))"></xeblog-hero>

[RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) is one of the most
widely deployed encryption algorithms in the world. Notably, when you generate
an SSH key without any extra flags, `ssh-keygen` will default to using RSA:

```
root@hiro:~# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
```

For a while cryptographers have feared that RSA is vulnerable to a quantum
computing algorithm known as [Shor's
Algorithm](https://en.wikipedia.org/wiki/Shor%27s_algorithm). I won't pretend to
understand it in this article, but the main reason why it's not deployed is that
the hardware required to attack RSA keys in the wild literally doesn't exist
yet (think literally tens of generations more advanced than current quantum
computers).

A group of researchers have just published [a
paper](https://arxiv.org/pdf/2212.12372.pdf) that posits that it's likely you
can break 2048-bit RSA (the most widely deployed keysize) with a quantum
computer that only uses 372 qubits of computational power. The [IBM
Osprey](https://newsroom.ibm.com/2022-11-09-IBM-Unveils-400-Qubit-Plus-Quantum-Processor-and-Next-Generation-IBM-Quantum-System-Two)
has 433 qubits.

<xeblog-conv name="Cadey" mood="coffee">Note that quantum computers are
effectively unobtainable (unless you're a research institution or you have a few
small loans of billions of dollars laying around), require a team of highly
specialized experts to monitor them 24/7, and aren't really usable to the
general public. I highly doubt that quantum computers are going to be rolling
into store shelves any time soon. I also have no idea what I'm talking about
with quantum computers. Please temper your interpretations of my statements
appropriately.</xeblog-conv>

It may be a good time to move away from RSA keys when and where you can. Today
I'm going to cover how to make SSH keys using
[ed25519](https://en.wikipedia.org/wiki/EdDSA#Ed25519) keys instead of RSA. 

<xeblog-conv name="Mara" mood="hacker">It's worth noting that RSA has not been
broken yet, the paper in question describes a theoretical attack. Quantum
computers are nowhere near good enough for this yet.</xeblog-conv>

## Generating new keys

To generate a new keypair, use the `ssh-keygen` command:

```
ssh-keygen -t ed25519
```

Make sure to set a password on that key and then you can add it to your SSH
agent with `ssh-add`. Copy the public key to your clipboard (print it to the
screen with `cat ~/.ssh/id_ed25519.pub`) and then you can add it to GitHub or
other services you use.

<xeblog-conv name="Mara" mood="hacker">Pro tip: you can get a list of machines
you've SSHed into by reading your `~/.ssh/known_hosts` file. You could use a
command like this:</xeblog-conv>

```
cat ~/.ssh/known_hosts | cut -d' ' -f1 | sort | uniq
```

<xeblog-conv name="Mara" mood="happy">This will get you a list of machines that
you may need to update your SSH key in! Remember that your new key should go to
the end of `~/.ssh/authorized_keys`!</xeblog-conv>

## Disabling RSA host keys

The OpenSSH server will create a keypair for each machine it runs on. By default
this creates an RSA key as well as an ed25519 key. You can disable this by
adding the following line to `/etc/ssh/sshd_config`:

```
HostKey /etc/ssh/ssh_host_ed25519_key
```

<xeblog-conv name="Mara" mood="hacker">In my testing, this was the case for both
NixOS and Ubuntu. If you want to be sure you're setting the right key, check the
file for commented-out HostKey instructions. Uncomment whichever one contains
`ed25519` in it.</xeblog-conv>

If your SSH configuration file has a `Ciphers`, `HostKeyAlgorithms`,
`PubkeyAcceptedAlgorithms`, or `CASignatureAlgorithms` setting in it, you may
want to make sure that any `rsa` cipher or algorithm isn't present in any of
them. If your distro has an option to change this system wide (such as in [Red
Hat and
derivatives](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/security_hardening/using-the-system-wide-cryptographic-policies_security-hardening)),
you may want to use that.

<xeblog-conv name="Mara" mood="happy">You may want to have some kind of
transition period for shared machines before you start rejecting RSA keys
willy-nilly. This can break people's workflows and SSH-key-in-GPG setups. Talk
with your users and work on compromises. Something something shill for the
company supplying the author of this post with the money needed for food
something something.</xeblog-conv>

If you want to do this on NixOS, add the following configuration to either your
`configuration.nix` or something that is imported by your `configuration.nix`:

```nix
services.openssh.hostKeys = [{
  path = "/etc/ssh/ssh_host_ed25519_key";
  type = "ed25519";
}];
```

<xeblog-conv name="Mara" mood="hacker">This tells SSH to use only an ed25519
host key. By default it will also create an RSA key.</xeblog-conv>

---

I hope this helps! Systems administration is full of annyoing migrations and
compromises like this. Good luck out there!

<xeblog-conv name="Mara" mood="hacker">Also check out [this
article](https://xeiaso.net/blog/yubikey-ssh-key-storage) on how you can store
an SSH key on a Yubikey or any other compliant FIDO2 key!</xeblog-conv>
