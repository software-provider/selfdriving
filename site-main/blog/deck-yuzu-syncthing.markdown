---
title: Syncing my emulator saves with Syncthing
date: 2023-05-26
---

<xeblog-conv name="Aoi" mood="wut">Hey, can we have Steam cloud saves
for my Steam Deck to the PC so I can play my emulated games on the go
without losing progress?</xeblog-conv>
<xeblog-conv name="Cadey" mood="coffee">No, we have cloud saves at
home.</xeblog-conv>

Or: cloud saves at home

<xeblog-hero ai="Ligne Claire+Teyvat" file="portable-adventure" prompt="masterpiece, ligne claire, 1girl, green hair, green eyes, hoodie, flat colors, space needle, summer, landscape"></xeblog-hero>

One of the most common upsells in gaming is "cloud saves", or some
encrypted storage space with your console/platform manufacturer to
store the save files that games make. Normally Steam, PlayStation,
Xbox, Nintendo, and probably EA offer this as a service to customers
because it makes for a _much better_ customer experience as the
customer migrates between machines. Recently I started playing Dead
Space 2 on Steam again on a lark and I got to open my old Dead Space 2
save from college. It's _magic_ when it works.

However, you should know this blog well enough that we're going way
outside the normal/expected use cases in this case. Today I'm gonna
show you how to make cloud saves for emulated Switch games at home
using [Syncthing](https://syncthing.net/) and
[EmuDeck](https://www.emudeck.com/).

For this example I'm going to focus on the following games:

- The Legend of Zelda: Breath of the Wild
- The Legend of Zelda: Tears of the Kingdom

I own these two games on cartridge and I have dumped my copy of them
from cartridge using a hackable Switch.

<xeblog-toot url="https://pony.social/@cadey/110434991652438811"></xeblog-toot>

Here's the other things you will need:

- [Tailscale installed on the Steam
  Deck](https://gist.github.com/legowerewolf/1b1670457cfac9201ee9d67840952147)
  (technically optional, but it means you don't need to use a browser
  on the Deck)
- A Windows PC running
  [SyncTrayzor](https://github.com/canton7/SyncTrayzor/releases) or a
  Linux server running Syncthing (Tailscale will also help in the
  latter case for reasons that will become obvious)
- A hackable switch and legal copies of the games you wish to emulate

<xeblog-conv name="Mara" mood="hacker" standalone>Props to
[@legowerewolf](https://github.com/legowerewolf) for turning the
[first attempt at getting Tailscale on the
Deck](https://tailscale.com/blog/steam-deck/) into something a bit
more robust.</xeblog-conv>

First, set up Syncthing on your PC by either installing SyncTrayzor or
enabling it in NixOS with this family of settings:
[`services.syncthing.*`](https://search.nixos.org/options?channel=23.05&from=0&size=50&sort=relevance&type=packages&query=services.syncthing).

At a high level though, here's how to find the right folder with
Yuzu emulator saves:

- Open Yuzu
- Right-click on the game
- Choose Open Save Data Location
- Go up two levels

This folder will contain a bunch of sub-folders with title
identifiers. That is where the game-specific save data is located. On
my Deck this is the following:

```
/home/deck/Emulation/saves/yuzu/saves/0000000000000000/
```

Your random UUID will be different than mine will be. We will handle
this in a later step. Write this path down or copy it to your
scratchpad.

## Installing Syncthing on the Deck

Next, you will need to install Syncthing on your Deck. There are
several ways to do this, but I think the most direct way will be to
install it in your user folder as a [systemd user
service](https://wiki.archlinux.org/title/Systemd/User).

<xeblog-conv name="Mara" mood="hacker" standalone>User services are
one of the truly standout features of systemd. They allow you to have
the same benefits of systemd's service management but for things owned
by _you_. This lets you get mounts for things like your network
storage mounts, grouping with targets so that you only enable some
things while gaming, and more.</xeblog-conv>

SSH into your Deck and download the [most recent release of
Syncthing](https://syncthing.net/downloads/).

```
wget https://domain.tld/path/syncthing-linux-amd64-<version>.tar.gz
```

Extract it with `tar xf`:

```
tar xf syncthing-linux-amd64-<version>.tar.gz
```

Then enter the folder:

```
cd syncthing-linux-amd64-<version>
```

Make a folder in ~ called `bin`, this is where Syncthing will live:

```
mkdir -p ~/bin
```

Move the Syncthing binary to `~/bin`:

```
mv syncthing ~/bin/
```

Then create a Syncthing user service at
`~/.config/systemd/user/syncthing.service`:

```
[Unit]
Description=Syncthing - Open Source Continuous File Synchronization
Documentation=man:syncthing(1)
StartLimitIntervalSec=60
StartLimitBurst=4

[Service]
ExecStart=/home/deck/bin/syncthing serve --no-browser --no-restart --logflags=0
Restart=on-failure
RestartSec=1
SuccessExitStatus=3 4
RestartForceExitStatus=3 4

# Hardening
SystemCallArchitectures=native
MemoryDenyWriteExecute=true
NoNewPrivileges=true

# Elevated permissions to sync ownership (disabled by default),
# see https://docs.syncthing.net/advanced/folder-sync-ownership
#AmbientCapabilities=CAP_CHOWN CAP_FOWNER

[Install]
WantedBy=default.target
```

And then start it once to make the configuration file:

```
systemctl --user start syncthing.service
sleep 2
systemctl --user stop syncthing.service
```

<xeblog-conv name="Cadey" mood="enby" standalone>If you aren't using
Tailscale on your Deck, click [here](#postTailscale) to skip past this
step.</xeblog-conv>

Now we need to set up Tailscale Serve to point to Syncthing and let
Syncthing allow the Tailscale DNS name. Open the syncthing 
configuration file with vim, your favorite text editor:

```
vim ~/.config/syncthing/config.xml
```

In the `<gui>` element, add the following configuration:

```xml
<insecureSkipHostcheck>true</insecureSkipHostcheck>
```

<xeblog-conv name="Mara" mood="hacker" standalone>Normally this is a
somewhat bad idea, but we're going to be exposing things over
Tailscale so it doesn't matter as much. This check makes sure that the
HTTP Host header from your browser matches "localhost" so that only
someone sitting at that machine can access it.</xeblog-conv>

Next, configure Tailscale Serve with this command:

```
sudo tailscale serve https / http://127.0.0.1:8384
```

This will make every request to `https://yourdeck.tailnet.ts.net` go
directly to Syncthing.

<span id="postTailscale"></span>

Next, enable the Syncthing unit to automatically start on boot:

```
systemctl --user enable syncthing --now
```

<xeblog-conv name="Mara" mood="hacker" standalone>The `--now` flag
will also issue a `systemctl start` command for you!</xeblog-conv>

## Syncing the things

Once Syncthing is running on both machines, open Syncthing's UI on
your PC. You should have both devices open in the same screen for the
best effect (this is where Tailscale Serve helps).

You will need to pair the devices together. If Syncthing is running on
both machines, then choose "Add remote device" and select the device
that matches the identification on the other machine. You will need to
do this for both sides.

Once you do that, you need to configure syncing your save folder. Make
a new synced folder with the "Add Folder" button and it will open a
modal dialog box.

Give it a name and choose a memorable folder ID such as "yuzu-saves".
Copy the path from your scratchpad into the folder path.

Then make a new shared folder on your PC pointing to the same location
(two levels up from yorur game save folder found via Yuzu). Give it the
same name and ID, but change the path as needed.

Next, on your Deck's Syncthing edit that folder with the edit button
and change over to the Sharing tab. Select your PC and check it. Click
Save and then it will trigger a sync. This will copy all your data
between both machines.

If you want, you can also set up a sync for the Yuzu `nand` folder.
This will automatically sync over dumped firmware and game updates. I
do this so that this is more effortless for me, but your needs may
differ. Also feel free to set up syncing for other emulators like
Dolphin.

<xeblog-conv name="Numa" mood="delet" standalone>Cloud saves at
home!</xeblog-conv>
