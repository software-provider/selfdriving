---
title: Building Virtual Networks with Pulumi and Tailscale
date: 2023-01-11
tags:
 - pulumi
 - tailscale
skip_ads: true
---

<xeblog-conv name="Cadey" mood="enby">This was a
[workshop](https://www.pulumi.com/resources/building-virtual-networks-with-pulumi-and-tailscale/)
that I helped with so that people could learn how to glue Tailscale and
[Pulumi](https://www.pulumi.com/) (think Terraform but you can declare resources
in programming languages such as TypeScript instead of HCL) together by creating
a Tailscale subnet router to connect you to a VPC in AWS. I'm including the
speaking bits that I did for the talk, but most of what I was there for was to
help field questions about Tailscale. Internet streamer brain is a useful tool
when properly harnessed.</xeblog-conv>

<xeblog-video path="talks/pulumi-workshop-2023"></xeblog-video>

---

<xeblog-slide name="pulumi-workshop-2023/001" essential></xeblog-slide>

Tailscale is a networking tool that helps you connect your computers together
like they were on the same network to begin with. Tailscale is built on top of
WireGuard and lets you access your servers, internal services, or file shares
from anywhere you have Internet access.

<xeblog-slide name="pulumi-workshop-2023/002"></xeblog-slide>

Today we're going to cover these important parts of Tailscale by setting up a
new AWS VPC and some servers behind it:

<xeblog-slide name="pulumi-workshop-2023/003"></xeblog-slide>

Tailscale lets you share machines on your tailnet (Tailscale network) so that
you can access them remotely, no matter where you are on the planet. Write that
screenplay at Starbucks via remote desktop without having to muck with port
forwarding or risking everything by exposing the port to the public Internet.
Grab the missing bit of paperwork that immigration needs from your NAS while you
are at the airport. Tailscale makes it possible for you to forget that you were
away from your home or work networks to begin with.

<xeblog-slide name="pulumi-workshop-2023/004"></xeblog-slide>

Tailscale doesn't stop at sharing individual computers though, you can share any
existing network segment with your tailnet using subnet routing. Subnet routing
lets existing infrastructure such as a legacy VPC with all of the computers
you're too afraid to touch be accessed over Tailscale too. No more StrongSwan
required. This is also useful for connecting to remote devices like IoT devices
that you really don't want to open up to the public internet. You can do this
all without having to configure complicated firewall rules.

<xeblog-slide name="pulumi-workshop-2023/005"></xeblog-slide>

This isn't limited to existing private networks. You can set up your own
"privacy VPN" on top of Tailscale by setting up an exit node. An exit node is a
machine on your tailnet that can act as a subnet router _for the entire
internet_. This will let you access things that are geo-restricted like tax
software.

<xeblog-slide name="pulumi-workshop-2023/006"></xeblog-slide>

Tailscale doesn't stop there, there's SSH management, file sharing, an
ngrok-like tunnelling solution, and so much more.
  
I'll hand things back over to Josh so we can learn more about Pulumi.
