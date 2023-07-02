---
title: My Homelab NAS on NixOS
date: 2021-11-29
---

Recently my husband and I built a NAS to store our Plex library among other
things. Our home network has had an absurd abundance of slack compute laying
around (to the point that I am almost unsure what I am going to do with it all),
but the main thing that the network lacked was a good place to put a lot of
data. Previously each of our towers had been kitted with 2 4TB rotational drives
that we ended up sharing our media between. This worked out well enough, but it
was kind of inconvenient to just use the storage. Things had to be mediated
between Linux and Windows and it just became a whole mess.

Then the biggest breaking point happened when we started to both get into
playing roomscale VR with eachother on the internet. Most of the VR stuff works
on Windows as its primary target, and our home Plex server was on my tower when
it was booted into Linux. As I did more and more VR, rebooting into Linux for
our anime nights started to become a chore. So we decided to make a NAS that
we'd store all that stuff on as well as act as our Plex server so my tower could
do whatever it wanted.

[Pedantically, most of the VR stuff does work fine on Linux if your distro of
choice is NOT NixOS. It is kind of annoying.](conversation://Mara/hacker)

We settled on a machine we're calling `itsuki`, the 5th machine in our homelab.
`itsuki` has a hexacore i5 10600 like the rest of the lab, but differs in the
amount of ram at only 16 GB of ram. It is inside a Fractal Node 803 case and
currently lives under my desk. It runs NixOS like the rest of the homelab (save
`logos`, which runs Windows 11 and has an RTX 2060 in it for other
experimentation), and installing it was fairly painless.

`itsuki` has 4 8TB 7200 RPM drives in it, which when combined with raidz1 gives
me effectively 20 TB of redundant storage on top of 32 TB of raw storage. I have
a few parent datasets that I use for organizing this:

* `rpool/backup` - backups from my homelab and other servers get dumped here for
  long term storage. This dataset is due to be replicated to rsync.net.
* `rpool/local` - machine-local storage that should not be backed up to the
  cloud. This is mainly used for the Nix store because everything there can be
  recomputed as needed, making backups redundant.
* `rpool/safe` - Files that can be eligible for cloud backups and things that we
  would rather keep than lose. Our media library is here as well as our giant
  data folder imaginatively named `/data`. Some virtual machine disk images live
  here too as well as `/`, `/home`, `/srv` and `/var`.
  
[Fun fact: `itsuki` is named after the 5th quintuplet in the Nakano quintuplets
from the anime The Quintessential Quintuplets. Itsuki's name is a pun on the
number 5, which fits because the NAS is the 5th node in the
homelab.](conversation://Cadey/enby)

Installing NixOS was utterly painless. Using a combination of settings from the
Arch Linux wiki (seriously wish I could get a printed copy of that thing, it's
worth its weight in gold for how much weird arcane things you can learn from
it), the NixOS wiki and copying things off of a Synology box's samba
configuration file, I managed to trick everything into working and now all the
machines on our tailnet can access the data on the NAS without too much trouble.
Even iPhones and iPads thanks to the recent addition of SMB mounting on
iP{hone|ad}OS. It also works over Tailscale too, so I can get into the NAS'
files anywhere I have an internet connection.

Thanks to a docker-compose manifest for Transmission shoved into a WireGuard
network, I can also **legally acquire** certain kinds of animated media that
aren't available in Canada and manage how much I share back over Tailscale too.
Thanks to Tailscale's Let's Encrypt support I can also create a progressive web
app that lets me monitor Transmission on the go and give my NAS instructions to
**legally acquire** more of this media should I want to.

I originally wanted to make this a post about how I set up NixOS and everything,
but it wasn't actually that big of a deal to do it. It just worked first try
once I managed to trick the uber gamer Aorus motherboard to shut up about Secure
Boot and just let me boot off of a damn USB drive. Probably broke Windows
booting on that thing in the process, but honestly I'd be tempted to consider
that a security feature to protect the NAS data from Windows booting on the
machine.

This setup was boring, and honestly for the kind of thing that I was setting up,
that's very much a good thing.
