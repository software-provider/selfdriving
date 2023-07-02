---
title: "Anything can be a message queue if you use it wrongly enough"
date: 2023-06-04
tags:
 - aws
 - cursed
 - tuntap
 - satire
---

<div class="warning"><xeblog-conv name="Cadey" mood="coffee"
standalone>Hi, <span id="hnwarning">readers</span>! This post is
satire. Don't treat it as something that is viable for production
workloads. By reading this post you agree to never implement or use
this accursed abomination. This article is released to the public for
educational reasons. Please do not attempt to recreate any of the
absurd acts referenced here.</xeblog-conv></div>

<xeblog-hero ai="Ligne Claire" file="nihao-xiyatu" prompt="1girl, green hair, green eyes, landscape, hoodie, backpack, space needle"></xeblog-hero>

<script>
if (document.referrer.match(/news.ycombinator.com/)) {
  document.getElementById("hnwarning").innerText = "Hacker News users";
}
</script>

You may think that the world is in a state of relative peace. Things
look like they are somewhat stable, but reality couldn't be farther
from the truth. There is an enemy out there that transcends time,
space, logic, reason, and lemon-scented moist towelettes. That enemy
is a scourge of cloud costs that is likely the single reason why
startups die from their cloud bills when they are so young.

The enemy is [Managed NAT
Gateway](https://aws.amazon.com/blogs/aws/new-managed-nat-network-address-translation-gateway-for-aws/).
It is a service that lets you egress traffic from a VPC to the public
internet at $0.07 per gigabyte. This is something that is probably
literally free for them to run but ends up getting a huge chunk of
their customer's cloud spend. Customers don't even look too deep into
this because they just shrug it off as the cost of doing business.

This one service has allowed companies like [the duckbill
group](https://www.duckbillgroup.com/) to make _millions_ by showing
companies how to not spend as much on the cloud.

However, I think I can do one better. What if there was a _better_ way
for your own services? What if there was a way you could reduce that
cost for your own services by up to 700%? What if you could bypass
those pesky network egress costs yet still contact your machines over
normal IP packets?

<xeblog-conv name="Aoi" mood="coffee">Really, if you are trying to
avoid Managed NAT Gateway in production for egress-heavy workloads
(such as webhooks that need to come from a common IP address), you
should be using a [Tailscale](https://www.tailscale.com) [exit
node](https://tailscale.com/kb/1103/exit-nodes/) with a public
IPv4/IPv6 address attached to it. If you also attach this node to the
same VPC as your webhook egress nodes, you can basically recreate
Managed NAT Gateway at home. You also get the added benefit of 
encrypting your traffic further on the wire.<br /><br />This is the
only thing in this article that you can safely copy into your
production workloads.</xeblog-conv>

## Base facts

Before I go into more detail about how this genius creation works,
here's some things to consider:

When AWS launched originally, it had three services:

- [S3](https://en.wikipedia.org/wiki/Amazon_S3) - Object storage for
  cloud-native applications
- [SQS](https://en.wikipedia.org/wiki/Amazon_Simple_Queue_Service) - A
  message queue
- [EC2](https://en.wikipedia.org/wiki/Amazon_Elastic_Compute_Cloud) -
  A way to run Linux virtual machines somewhere
  
Of those foundational services, I'm going to focus the most on S3: the
Simple Storage Service. In essence, S3 is `malloc()` for the cloud.

<xeblog-conv name="Mara" mood="hacker" standalone>If you already know
what S3 is, please click [here](#postcloud) to skip this explanation.
It may be worth revisiting this if you do though!</xeblog-conv>

### The C programming language

When using the C programming language, you normally are working with
memory in the stack. This memory is almost always semi-ephemeral and
all of the contents of the stack are no longer reachable (and
presumably overwritten) when you exit the current function. You can do
many things with this, but it turns out that this isn't very useful in
practice. To work around this (and reliably pass mutable values
between functions), you need to use the
[`malloc()`](https://www.man7.org/linux/man-pages/man3/malloc.3.html)
function. `malloc()` takes in the number of bytes you want to allocate
and returns a pointer to the region of memory that was allocated.

<xeblog-conv name="Aoi" mood="sus">Huh? That seems a bit easy for C.
Can't allocating memory fail when there's no more free memory to
allocate? How do you handle that?</xeblog-conv>
<xeblog-conv name="Mara" mood="happy">Yes, allocating memory can
fail. When it does fail it returns a null pointer and sets the
[errno](https://www.man7.org/linux/man-pages/man3/errno.3.html)
superglobal variable to the constant `ENOMEM`. From here all behavior
is implementation-defined.</xeblog-conv>
<xeblog-conv name="Aoi" mood="coffee">Isn't "implementation-defined"
code for "it'll probably crash"?</xeblog-conv>
<xeblog-conv name="Mara" mood="hacker">In many cases: yes most of the
time it will crash. Hard. Some applications are smart enough to handle
this more gracefully (IE: try to free memory or run a garbage
collection run), but in many cases it doesn't really make more sense
to do anything but crash the program.</xeblog-conv>
<xeblog-conv name="Aoi" mood="facepalm">Oh. Good. Just what I wanted
to hear.</xeblog-conv>

When you get a pointer back from `malloc()`, you can store anything in
there as long as it's the same length as you passed or less.

<xeblog-conv name="Numa" mood="delet" standalone>Fun fact: if you
overwrite the bounds you passed to `malloc()` and anything involved in
the memory you are writing is user input, congradtulations: you just
created a way for a user to either corrupt internal application state
or gain arbitrary code execution. A similar technique is used in
The Legend of Zelda: Ocarina of Time speedruns in order to get
arbitrary code execution via [Stale Reference
Manipulation](https://www.zeldaspeedruns.com/oot/srm/srm-overview).</xeblog-conv>

Oh, also anything stored in that pointer to memory you got back from
`malloc()` is stored in an area of ram called "the heap", which is
moderately slower to access than it is to access the stack.

### S3 in a nutshell

Much in the same way, S3 lets you allocate space for and submit
arbitrary bytes to the cloud, then fetch them back with an address.
It's a lot like the `malloc()` function for the cloud. You can put
bytes there and then refer to them between cloud functions.

<xeblog-conv name="Mara" mood="hacker" standalone>The bytes are stored
in the cloud, which is slightly slower to read from than it would be
to read data out of the heap.</xeblog-conv>

And these arbitrary bytes can be _anything_. S3 is usually used for
hosting static assets (like all of the conversation snippet avatars
that a certain website with an orange background hates), but nothing
is stopping you from using it to host literally anything you want.
Logging things into S3 is so common it's literally a core product
offering from Amazon. Your billing history goes into S3. If you
download your tax returns from WealthSimple, it's probably downloading
the PDF files from S3. VRChat avatar uploads and downloads are done
via S3.

<xeblog-conv name="Mara" mood="happy" standalone>It's like an FTP
server but you don't have to care about running out of disk space on
the FTP server!</xeblog-conv>

### IPv6

You know what else is bytes? [IPv6
packets](https://en.wikipedia.org/wiki/IPv6_packet). When you send an
IPv6 packet to a destination on the internet, the kernel will prepare
and pack a bunch of bytes together to let the destination and
intermediate hops (such as network routers) know where the packet
comes from and where it is destined to go.

Normally, IPv6 packets are handled by the kernel and submitted to a
queue for a hardware device to send out over some link to the
Internet. This works for the majority of networks because they deal
with hardware dedicated for slinging bytes around, or in some cases
shouting them through the air (such as when you use Wi-Fi or a mobile
phone's networking card).

<xeblog-conv name="Aoi" mood="coffee">Wait, did you just say that
Wi-Fi is powered by your devices shouting at eachother?</xeblog-conv>
<xeblog-conv name="Cadey" mood="aha">Yep! Wi-Fi signal strength is
measured in decibels even!</xeblog-conv>
<xeblog-conv name="Numa" mood="delet">Wrong. Wi-Fi is more accurately
_light_, not _sound_. It is much more accurate to say that the devices
are _shining_ at eachother. Wi-Fi is the product of radio waves, which
are the same thing as light (but it's so low frequency that you can't
see it). Boom. Roasted.</xeblog-conv>

### The core Unix philosophy: everything is a file

<span id="postcloud"></span>
There is a way to bypass this and have software control how network
links work, and for that we need to think about Unix conceptually for
a second. In the hardcore Unix philosophical view: everything is a
file. Hard drives and storage devices are files. Process information
is viewable as files. Serial devices are files. This core philosophy
is rooted at the heart of just about everything in Unix and Linux
systems, which makes it a lot easier for applications to be
programmed. The same API can be used for writing to files, tape
drives, serial ports, and network sockets. This makes everything a lot
conceptually simpler and reusing software for new purposes trivial.

<xeblog-conv name="Mara" mood="hacker" standalone>As an example of
this, consider the
[`tar`](https://man7.org/linux/man-pages/man1/tar.1.html) command. The
name `tar` stands for "Tape ARchive". It was a format that was created
for writing backups [to actual magnetic tape
drives](https://en.wikipedia.org/wiki/Tape_drive). Most commonly, it's
used to download source code from GitHub or as an interchange format
for downloading software packages (or other things that need to put
multiple files in one distributable unit).</xeblog-conv>

In Linux, you can create a
[TUN/TAP](https://en.wikipedia.org/wiki/TUN/TAP) device to let
applications control how network or datagram links work. In essence,
it lets you create a file descriptor that you can read packets from
and write packets to. As long as you get the packets to their intended
destination somehow and get any other packets that come back to the
same file descriptor, the implementation isn't relevant. This is how
OpenVPN, ZeroTier, FreeLAN, Tinc, Hamachi, WireGuard and Tailscale
work: they read packets from the kernel, encrypt them, send them to
the destination, decrypt incoming packets, and then write them back
into the kernel.

### In essence

So, putting this all together:

* S3 is `malloc()` for the cloud, allowing you to share arbitrary
  sequences of bytes between consumers.
* IPv6 packets are just bytes like anything else.
* TUN devices let you have arbitrary application code control how
  packets get to network destinations.

In theory, all you'd need to do to save money on your network bills
would be to read packets from the kernel, write them to S3, and then
have another loop read packets from S3 and write those packets back
into the kernel. All you'd need to do is wire things up in the right
way.

So I did just that.

Here's some of my friends' reactions to that list of facts:

- I feel like you've just told me how to build a bomb. I can't belive
  this actually works but also I don't see how it wouldn't. This is
  evil.
- It's like using a warehouse like a container ship. You've put a
  warehouse on wheels.
- I don't know what you even mean by that. That's a storage method.
  Are you using an extremely generous definition of "tunnel"?
- sto psto pstop stopstops
- We play with hypervisors and net traffic often enough that we know
  that this is something someone wouldn't have thought of.
- Wait are you planning to actually *implement and use* ipv6 over
  s3?
- We're paying good money for these shitposts :)
- Is routinely coming up with cursed ideas a requirement for working
  at tailscale?
- That is horrifying. Please stop torturing the packets. This is a
  violation of the Geneva Convention.
- Please seek professional help.

<xeblog-conv name="Cadey" mood="enby" standalone>Before any of you
ask, yes, this was the result of a drunken conversation with [Corey
Quinn](https://twitter.com/quinnypig).</xeblog-conv>

## Hoshino

Hoshino is a system for putting outgoing IPv6 packets into S3 and then
reading incoming IPv6 packets out of S3 in order to avoid the absolute
dreaded scourge of Managed NAT Gateway. It is a travesty of a tool
that does work, if only barely.

The name is a reference to the main character of the anime [Oshi no
Ko](https://en.wikipedia.org/wiki/Oshi_no_Ko), Hoshino Ai. Hoshino is
an absolute genius that works as a pop idol for the group B-Komachi.

Hoshino is a shockingly simple program. It creates a TUN device,
configures the OS networking stack so that programs can use it, and
then starts up two threads to handle reading packets from the kernel
and writing packets into the kernel.

When it starts up, it creates a new TUN device named either `hoshino0`
or an administrator-defined name with a command line flag. This
interface is only intended to forward IPv6 traffic.

Each node derives its IPv6 address from the
[`machine-id`](https://www.man7.org/linux/man-pages/man5/machine-id.5.html)
of the system it's running on. This means that you can somewhat
reliably guarantee that every node on the network has a unique address
that you can easily guess (the provided ULA /64 and then the first
half of the `machine-id` in hex). Future improvements may include
publishing these addresses into DNS via Route 53.

When it configures the OS networking stack with that address, it uses
a [netlink](https://en.wikipedia.org/wiki/Netlink) socket to do this.
Netlink is a Linux-specific socket family type that allows userspace
applications to configure the network stack, communicate to the
kernel, and communicate between processes. Netlink sockets cannot
leave the current host they are connected to, but unlike Unix sockets
which are addressed by filesystem paths, Netlink sockets are addressed
by process ID numbers.

In order to configure the `hoshino0` device with Netlink, Hoshino does
the following things:

- Adds the node's IPv6 address to the `hoshino0` interface
- Enables the `hoshino0` interface to be used by the kernel
- Adds a route to the IPv6 subnet via the `hoshino0` interface

Then it configures the AWS API client and kicks off both of the main
loops that handle reading packets from and writing packets to the
kernel.

When uploading packets to S3, the key for each packet is derived from
the destination IPv6 address (parsed from outgoing packets using the
handy library
[gopacket](https://pkg.go.dev/github.com/google/gopacket)) and the
packet's unique ID (a
[ULID](https://pkg.go.dev/github.com/oklog/ulid/v2) to ensure that
packets are lexicographically sortable, which will be important to
ensure in-order delivery in the other loop).

When packets are processed, they are added to a
[bundle](https://pkg.go.dev/within.website/x/internal/bundler) for
later processing by the kernel. This is relatively boring code and
understanding it is mostly an exercise for the reader. `bundler` is
based on the Google package
[`bundler`](https://pkg.go.dev/google.golang.org/api/support/bundler),
but modified to use generic types because the original
implementation of `bundler` predates them.

### cardio

However, the last major part of understanding the genius at play here
is by the use of [cardio](https://pkg.go.dev/within.website/x/cardio).
Cardio is a utility in Go that lets you have a "heartbeat" for events
that should happen every so often, but also be able to influence the
rate based on need. This lets you speed up the rate if there is more
work to be done (such as when packets are found in S3), and reduce the
rate if there is no more work to be done (such as when no packets are
found in S3).

<xeblog-conv name="Aoi" mood="coffee" standalone>Okay, this is also
probably something that you can use outside of this post, but I
promise there won't be any more of these!</xeblog-conv>

When using cardio, you create the heartbeat channel and signals like
this:

```go
heartbeat, slower, faster := cardio.Heartbeat(ctx, time.Minute, time.Millisecond)
```

The first argument to `cardio.Heartbeat` is a
[`context`](https://pkg.go.dev/context) that lets you cancel the
heartbeat loop. Additionally, if your application uses
[`ln`](https://xeiaso.net/blog/ln-the-natural-logger-2020-10-17)'s
[`opname`](https://pkg.go.dev/within.website/ln/opname) facility, an
[`expvar`](https://pkg.go.dev/expvar) gauge will be created and named
after that operation name.

The next two arguments are the minimum and maximum heart rate. In this
example, the heartbeat would range between once per minute and once
per millisecond.

When you signal the heart rate to speed up, it will double the rate.
When you trigger the heart rate to slow down, it will halve the rate.
This will enable applications to spike up and gradually slow down as
demand changes, much like how the human heart will speed up with
exercise and gradually slow down as you stop exercising.

When the heart rate is too high for the amount of work needed to be
done (such as when the heartbeat is too fast, much like tachycardia in
the human heart), it will automatically back off and signal the heart
rate to slow down (much like I wish would happen to me sometimes).

This is a package that I always wanted to have exist, but never found
the need to write for myself until now.

### Terraform

Like any good recovering SRE, I used
[Terraform](https://www.terraform.io/) to automate creating
[IAM](https://aws.amazon.com/iam/) users and security policies for
each of the nodes on the Hoshino network. This also was used to create
the S3 bucket. Most of the configuration is fairly boring, but I did
run into an issue while creating the policy documents that I feel is
worth pointing out here.

I made the "create a user account and policies for that account" logic
into a Terraform module because that's how you get functions in
Terraform. It looked like this:

```hcl
data "aws_iam_policy_document" "policy" {
  statement {
    actions = [
      "s3:GetObject",
      "s3:PutObject",
      "s3:ListBucket",
    ]
    effect = "Allow"
    resources = [
      var.bucket_arn,
    ]
  }

  statement {
    actions   = ["s3:ListAllMyBuckets"]
    effect    = "Allow"
    resources = ["*"]
  }
}
```

When I tried to use it, things didn't work. I had given it the
permission to write to and read from the bucket, but I was being told
that I don't have permission to do either operation. The reason this
happened is because my statement allowed me to put objects to the
bucket, but not to any path INSIDE the bucket. In order to fix this, I
needed to make my policy statement look like this:

```hcl
statement {
  actions = [
    "s3:GetObject",
    "s3:PutObject",
    "s3:ListBucket",
  ]
  effect = "Allow"
  resources = [
    var.bucket_arn,
    "${var.bucket_arn}/*", # allow every file in the bucket
  ]
}
```

This does let you do a few cool things though, you can use this to
create per-node credentials in IAM that can only write logs to their
part of the bucket in particular. I can easily see how this can be
used to allow you to have infinite flexibility in what you want to do,
but good lord was it inconvenient to find this out the hard way.

Terraform also configured the lifecycle policy for objects in the
bucket to delete them after a day.

```hcl
resource "aws_s3_bucket_lifecycle_configuration" "hoshino" {
  bucket = aws_s3_bucket.hoshino.id

  rule {
    id = "auto-expire"

    filter {}

    expiration {
      days = 1
    }

    status = "Enabled"
  }
}
```

<xeblog-conv name="Cadey" mood="coffee" standalone>If I could, I would
set this to a few hours at most, but the minimum granularity for S3
lifecycle enforcement is in days. In a loving world, this should be a
sign that I am horribly misusing the product and should stop. I did
not stop.</xeblog-conv>

### The horrifying realization that it works

Once everything was implemented and I fixed the last bugs related to
[the efforts to make Tailscale faster than kernel
wireguard](https://tailscale.com/blog/more-throughput/), I tried to
ping something. I set up two virtual machines with
[waifud](https://xeiaso.net/blog/series/waifud) and installed Hoshino.
I configured their AWS credentials and then started it up. Both
machines got IPv6 addresses and they started their loops. Nervously, I
ran a ping command:

```
xe@river-woods:~$ ping fd5e:59b8:f71d:9a3e:c05f:7f48:de53:428f
PING fd5e:59b8:f71d:9a3e:c05f:7f48:de53:428f(fd5e:59b8:f71d:9a3e:c05f:7f48:de53:428f) 56 data bytes
64 bytes from fd5e:59b8:f71d:9a3e:c05f:7f48:de53:428f: icmp_seq=1 ttl=64 time=2640 ms
64 bytes from fd5e:59b8:f71d:9a3e:c05f:7f48:de53:428f: icmp_seq=2 ttl=64 time=3630 ms
64 bytes from fd5e:59b8:f71d:9a3e:c05f:7f48:de53:428f: icmp_seq=3 ttl=64 time=2606 ms
```

It worked. I successfully managed to send ping packets over Amazon S3.
At the time, I was in an airport dealing with the aftermath of [Air
Canada's IT system falling the heck
over](https://www.cbc.ca/news/business/air-canada-outage-1.6861923)
and the sheer feeling of relief I felt was better than drugs.

<xeblog-conv name="Cadey" mood="coffee" standalone>Sometimes I wonder
if I'm an adrenaline junkie for the unique feeling that you get when
your code finally works.</xeblog-conv>

Then I tested TCP. Logically holding, if ping packets work, then TCP
should too. It would be slow, but nothing in theory would stop it. I
decided to test my luck and tried to open the other node's metrics
page:

```
$ curl http://[fd5e:59b8:f71d:9a3e:c05f:7f48:de53:428f]:8081
# skipping expvar "cmdline" (Go type expvar.Func returning []string) with undeclared Prometheus type
go_version{version="go1.20.4"} 1
# TYPE goroutines gauge
goroutines 208
# TYPE heartbeat_hoshino.s3QueueLoop gauge
heartbeat_hoshino.s3QueueLoop 500000000
# TYPE hoshino_bytes_egressed gauge
hoshino_bytes_egressed 3648
# TYPE hoshino_bytes_ingressed gauge
hoshino_bytes_ingressed 3894
# TYPE hoshino_dropped_packets gauge
hoshino_dropped_packets 0
# TYPE hoshino_ignored_packets gauge
hoshino_ignored_packets 98
# TYPE hoshino_packets_egressed gauge
hoshino_packets_egressed 36
# TYPE hoshino_packets_ingressed gauge
hoshino_packets_ingressed 38
# TYPE hoshino_s3_read_operations gauge
hoshino_s3_read_operations 46
# TYPE hoshino_s3_write_operations gauge
hoshino_s3_write_operations 36
# HELP memstats_heap_alloc current bytes of allocated heap objects (up/down smoothly)
# TYPE memstats_heap_alloc gauge
memstats_heap_alloc 14916320
# HELP memstats_total_alloc cumulative bytes allocated for heap objects
# TYPE memstats_total_alloc counter
memstats_total_alloc 216747096
# HELP memstats_sys total bytes of memory obtained from the OS
# TYPE memstats_sys gauge
memstats_sys 57625662
# HELP memstats_mallocs cumulative count of heap objects allocated
# TYPE memstats_mallocs counter
memstats_mallocs 207903
# HELP memstats_frees cumulative count of heap objects freed
# TYPE memstats_frees counter
memstats_frees 176183
# HELP memstats_num_gc number of completed GC cycles
# TYPE memstats_num_gc counter
memstats_num_gc 12
process_start_unix_time 1685807899
# TYPE uptime_sec counter
uptime_sec 27
version{version="1.42.0-dev20230603-t367c29559-dirty"} 1
```

I was floored. It works. The packets were sitting there in S3, and I
was able to pluck out [the TCP
response](https://cdn.xeiaso.net/file/christine-static/blog/2023/hoshino/01H20ZQ3H9CW1FS9CAX6JX0NPY)
and I opened it with `xxd` and was able to confirm the source and
destination address:

```
00000000: 6007 0404 0711 0640
00000008: fd5e 59b8 f71d 9a3e
00000010: c05f 7f48 de53 428f
00000018: fd5e 59b8 f71d 9a3e
00000020: 59e5 5085 744d 4a66
```

It was `fd5e:59b8:f71d:9a3e:59e5:5085:744d:4a66` trying to reach
`fd5e:59b8:f71d:9a3e:c05f:7f48:de53:428f`.

<xeblog-conv name="Aoi" mood="wut">Wait, if this is just putting
stuff into S3, can't you do deep packet inspection with Lambda [by
using the workflow for automatically generating
thumbnails](https://docs.aws.amazon.com/lambda/latest/dg/with-s3-example.html)?</xeblog-conv>
<xeblog-conv name="Numa" mood="happy">Yep! This would let you do it
fairly trivially even. I'm not sure how you would prevent things from
getting through, but you could have your lambda handler funge a TCP
packet to either side of the connection with [the `RST` flag
set](https://www.rfc-editor.org/rfc/rfc793.html#section-3.1:~:text=Reset%20Generation%0A%0A%20%20As%20a%20general%20rule%2C%20reset%20(RST)%20must%20be%20sent%20whenever%20a%20segment%20arrives%0A%20%20which%20apparently%20is%20not%20intended%20for%20the%20current%20connection.%20%20A%20reset%0A%20%20must%20not%20be%20sent%20if%20it%20is%20not%20clear%20that%20this%20is%20the%20case.)
(RFC 793: Transmission Control Protocol, the RFC that defines TCP,
page 36, section "Reset Generation"). That could let you kill
connections that meet unwanted criteria, at the cost of having to
invoke a lambda handler. I'm _pretty sure_ this is RFC-compliant, but
I'm a shitposter, not a the network police.</xeblog-conv>
<xeblog-conv name="Aoi" mood="wut">Oh. I see.<br /><br />Wait, how did
you have 1.8 kilobytes of data in that packet? Aren't packets usually
smaller than that?</xeblog-conv>
<xeblog-conv name="Mara" mood="happy">When dealing with networking
hardware, you can sometimes get _frames_ (the networking hardware
equivalent of a packet) to be up to 9000 bytes with [jumbo
frames](https://en.wikipedia.org/wiki/Jumbo_frame), but if your
hardware does support jumbo frames then you can usually get away with
9216 bytes at max.</xeblog-conv>
<xeblog-conv name="Numa" mood="delet">It's over nine-</xeblog-conv>
<xeblog-conv name="Mara" mood="hacker">Yes dear, it's over 9000. Do
keep in mind that we aren't dealing with physical network equipment
here, so realistically our packets can be up to to the limit of the
IPv6 packet header format: the oddly specific number of 65535 bytes.
This is configured by the Maximum Transmission Unit at the OS level
(though usually this defines the limit for network frames and not IP
packets). Regardless, Hoshino defaults to an MTU of 53049, which
should allow you to transfer a bunch of data in a single S3
object.</xeblog-conv>

## Cost analysis

When you count only network traffic costs, the architecture has many
obvious advantages. Access to S3 is zero-rated in many cases with S3,
however the real advantage comes when you are using this cross-region.
This lets you have a worker in us-east-1 communicate with another
worker in us-west-1 without having to incur the high bandwidth cost
per gigabyte when using Managed NAT Gateway.

However, when you count all of the S3 operations (up to one every
millisecond), Hoshino is hilariously more expensive because of simple
math you can do on your own napkin at home.

For the sake of argument, consider the case where an idle node is
sitting there and polling S3 for packets. This will happen at the
minimum poll rate of once every 500 milliseconds. There are 24 hours
in a day. There are 60 minutes in an hour. There are 60 seconds in a
minute. There are 1000 milliseconds in a second. This means that each
node will be making 172,800 calls to S3 per day, at a cost of $<span id="hnprice1">0.86</span>
per node per day. And that's what happens with no traffic. When
traffic happens that's at least one additional `PUT`-`GET` call pair
_per-packet_.

Depending on how big your packets are, this can cause you to easily
triple that number, making you end up with 518,400 calls to S3 per day
($<span id="hnprice2">2.59</span> per node per day). Not to mention
TCP overhead from the three-way handshake and acknowledgement packets.

This is hilariously unviable and makes the effective cost of
transmitting a gigabyte of data over HTTP through such a contraption
vastly more than $0.07 per gigabyte.

<script>
if (document.referrer.match(/news.ycombinator.com/) || localStorage["hn_gaslight_multiplicand"]) {
  const price1Base = 0.86;
  const price2Base = 2.59;
  
  const multiplicand = localStorage["hn_gaslight_multiplicand"] || Math.random() * (5 - 0.8) + 0.8;

  localStorage["hn_gaslight_multiplicand"] = multiplicand;
  
  document.getElementById("hnprice1").innerText = `${(price1Base * multiplicand).toFixed(2)}`;
  document.getElementById("hnprice2").innerText = `${(price2Base * multiplicand).toFixed(2)}`;
}
</script>

## Other notes

This architecture does have a strange advantage to it though: assuming
a perfectly spherical cow, adequate network latency, and sheer luck
this does make UDP a bit more reliable than it should be otherwise.

With appropriate timeouts and retries at the application level, it may
end up being more reliable than IP transit over the public internet.

<xeblog-conv name="Aoi" mood="coffee" standalone>Good lord is this an
accursed abomination.</xeblog-conv>

I guess you could optimize this by replacing the S3 read loop with
some kind of AWS lambda handler that remotely wakes the target
machine, but at that point it may actually be better to have that
lambda POST the contents of the packet to the remote machine. This
would let you bypass the S3 polling costs, but you'd still have to pay
for the egress traffic from lambda and the posting to S3 bit.

<xeblog-conv name="Cadey" mood="coffee" standalone>Before you comment
about how I could make it better by doing x, y, or z; please consider
that I need to leave room for a part 2. I've already thought about
nearly anything you could have already thought about, including using
SQS, bundling multiple packets into a single S3 object, and other
things that I haven't mentioned here for brevity's sake.</xeblog-conv>

## Shitposting so hard you create an IP conflict

Something amusing about this is that it is something that technically
steps into the realm of things that my employer does. This creates a
unique kind of conflict where I can't easily retain the intellectial
property (IP) for this without getting it approved from my employer.
It is a bit of the worst of both worlds where I'm doing it on my own
time with my own equipment to create something that will be ultimately
owned by my employer. This was a bit of a sour grape at first and I
almost didn't implement this until the whole Air Canada debacle
happened and I was very bored.

However, I am choosing to think about it this way: I have successfully
shitposted so hard that it's a legal consideration and that I am going
to be _absolved of the networking sins I have committed_ by instead
outsourcing those sins to my employer.

I was told that under these circumstances I could release the source
code and binaries for this atrocity (provided that I release them with
the correct license, which I have rigged to be included in both the
source code and the binary of Hoshino), but I am going to elect to not
let this code see the light of day outside of my homelab. Maybe I'll
change my mind in the future, but honestly this entire situation is so
cursed that I think it's better for me to not for the safety of
humankind's minds and wallets.

I may try to use the basic technique of Hoshino as a replacement for
DERP, but that sounds like a lot of effort after I have proven that
this is so hilariously unviable. It would work though!

---

<xeblog-conv name="Aoi" mood="grin">This would make a great
[SIGBOVIK](http://sigbovik.org/) paper.</xeblog-conv>
<xeblog-conv name="Cadey" mood="enby">Stay tuned. I have
plans.</xeblog-conv>
