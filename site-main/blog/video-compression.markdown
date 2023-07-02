---
title: Video Compression for Mere Mortals
date: 2023-02-08
tags:
 - ffmpeg
 - scalability
 - xedn
 - nix
---

I'm not just an internet streamer, I'm a
[VTuber](https://xeiaso.net/blog/vtubing-setup-2022-01-13). I end up
streaming every other weekend [on
Twitch](https://twitch.tv/princessxen) where I work on a variety of
things and attempt to explain my process as I am doing them. I started
doing this as a way to help be more comfortable with public speaking
and it has been _absolutely catalytic_ to get over that fear.

This website is intended to be my professional portfolio, and I
haven't really had a good way to encode things like the events I've
attended or the streams I have done. As I progress deeper and deeper
into the craft of developer relations, these things are becoming more
obvious and I need something like this.

<xeblog-conv standalone name="Cadey" mood="coffee">To be fair, I _do_
have the ability to annotate individual posts with links to the
relevant stream recordings, but that is a very different thing than a
list of streams that have been done and an outline of what happened in
them. Those links are also at the bottom of the page, and I suspect
that nobody ends up reading those or clicking through to
them.</xeblog-conv>

However, just making the section with a YouTube embed would work, but
I don't feel that's in the true spirit of the Xe Iaso brand. If I have
a section for streams on my website, I want people to be able to
_watch those streams_ on my website. I don't want to be limited by how
YouTube or Twitch compresses videos or limits the use of
picture-in-picture. I want to self-host the videos.

Unfortunately, there are a few problems with this:

- My stream source files are _gigabytes_, at about 2 gigabytes of
  video per hour of footage. Based on some rough back-of-the-napkin
  estimates, it would be vastly uneconomical to serve unoptimized
  video, even though my embedded video files have a very low
  click-through-rate (about 40% in my estimates from Prometheus).
- I messed up with the fundamental architecture of
  [XeDN](https://xeiaso.net/blog/xedn) and part of how it caches files
  involves reading those files into ram. The XeDN nodes on fly.io only
  have 256MB of ram each. I could change how this works to make that
  easier, but that would be a lot of architecting and engineering that
  I don't really want to have to think about. XeDN is a very simple
  thing and it is really easy to keep all of the constraints and
  advantages of it in my head. I don't want to lose that.
- XeDN currently is powered by 5 GB data volumes, where the number was
  chosen arbitrarily so that I wouldn't have to think about the
  problem for a year or two. I could increase the size of the volumes,
  but I don't want to think about that yet.
- The way I distribute videos with
  [HLS](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) already
  works _amazingly_ with how XeDN's core architecture works. I want to
  make this better instead of having to vastly reinvent the
  fundamental architecture of all the moving parts.

So, I'm gonna need to compress the video a bit better. Before I set
out on this journey, my best efforts got me about 1 gigabyte per hour
of video. After doing more back-of-the-napkin bandwidth and storage
math, I was okay with that. I would have to scale things up a bit
sooner than I had hoped, but this is the natural consequence of making
things more elaborate. Thanks to help from [a
friend](https://artemis.sh), I figured out how to compress my video a
lot. It's all compressed down to 330 MB for a 3.5 hour video. With the
fact that I do about 2-4 hours of streaming every two weeks, this
means I would need to think about the storage demands of XeDN in about
June. This is acceptable for me. I'm keeping an eye on the disk usage
of the XeDN volumes and have an alert set up in case there's a huge
spike in disk usage somehow, but it should be fine until then.

<xeblog-conv name="Aoi" mood="wut">How do you have so many
napkins?</xeblog-conv>

<xeblog-conv name="Cadey" mood="enby">I get the bulk packs from
Costco. Still working through the second of four packs of 1.3 thousand
napkins. I'm sure I'll get there soon!</xeblog-conv>

<xeblog-hero ai="Pastel-mix" file="sakura-onsen" prompt="landscape, spring, onsen, highly detailed, colorful, cherry blossoms, watercolor, no humans"></xeblog-hero>

## Video compression

In case you've never been down this rabbit hole, let's take a moment
to cover the basic problems that video compression solves. At a high
level, video is a bunch of pictures that is played back fast enough
that it tricks your brain into thinking that there is motion.

<xeblog-conv name="Mara" mood="hacker" standalone>Fun fact: what we
call video used to be called "moving pictures" when it was first
introduced. Over time people sanded away at the words and this lead to
the term "movies" to describe them. This is the origin of the term
"movie" as we know it today.</xeblog-conv>

A naïve way to handle this would be to just take each frame of video
and store it on the disk with a little bit of compression. This is how
video is stored in many professional formats like RED raw and Apple's
ProRes video, but that unmatched level of quality comes with an
unmatched level of disk space usage. One minute of 4K ProRes video can
easily eat up six whole gigabytes of space. This level of quality is
_absolutely required_ for professional video editing because of the
flexibility it gives you, but for most other uses (such as making
stream recordings of me being very bad at using CSS) it's vastly
overkill.

<xeblog-conv name="Mara" mood="hacker">The main reason why
professionals want the flexibility of raw video formats is that every
time you encode video into a codec like h.264 (the main video codec
used on the internet, usually in .mp4 files) you lose some of the
quality permanently. For an example of this in action, see
[deepfriedmemes.com](https://deepfriedmemes.com), which allows you to
see the generational compression loss on images using JPEG compression
as an artform for creating sarcastic versions of internet posts. I've
included an example of this below:</xeblog-conv>

| Before generational loss | After generational loss |
|---------|---------|
| <xeblog-picture path="blog/video-compression/before"></xeblog-picture> | <xeblog-picture path="blog/video-compression/after"></xeblog-picture> |

<xeblog-conv name="Mara" mood="happy">You can understand why this
would be a bad thing to have to deal with in a professional
environment! This is a bit of a bad example of this in particular
because both the source images were stored as jpeg files before they
were ingested by XeDN and if you look closely you can already see
faint signs of generational loss around the borders of the image on
the left. The image on the left was generated using [CDI
Diffusion](https://civitai.com/models/6020/zelda-cdi-style-lora).</xeblog-conv>

There is a simple way to compress all of this though, and for that
we're going to have to look at animation like Disney did it back when
they were still animating everything by hand.

When Disney movies were drawn by hand, they had two tiers of artists:
key frame artists, and in-between artists. The key frame artists drew
entire base images (or, more realistically, individual layers of those
things that were manually composited together at time of filming) for
key parts of single scenes and then the rest of the work was done by
the in-between frame artists. People again sanded away on the terms
and the fully composited images became known as key frames and the
rest were known as tween frames.

<xeblog-conv standalone name="Mara" mood="hacker">A good example of
how expensive it can be to have each frame painstakingly made by
experts is the anime movie
[REDLINE](https://en.wikipedia.org/wiki/Redline_(2009_film)). It is an
absolutely _fantastic_ creation, but took over 7 years to produce and
was a commercial failure. This almost took out the studio producing
it, but it is quite literally one of the most beautiful anime movies
ever made. It's up there with Miyazaki movies in how finely crafted it
is. It's well worth a watch.</xeblog-conv>

With modern video compression techniques, we can actually take
advantage of the fact that each frame in a video file is _almost
identical_ to the frame before it. So, instead of storing the entire
frame of video, we can only record the differences between that frame
and the previous one. This is the key discovery that lead to online
video streaming being viable before high-speed Internet was deployed
to and used by the masses. It's why we have Netflix.

<xeblog-conv standalone name="Mara" mood="hacker">There's an artform
called
[datamoshing](https://en.wikipedia.org/wiki/Compression_artifact#Artistic_use)
that involves using the tween frames from another video without the
correlating keyframes as an artistic effect. Consider this
meme:</xeblog-conv>

<xeblog-toot url="https://pony.social/@cadey/109813678329119959"></xeblog-toot>

## VTubing and compression

With this understanding of how video compression works (tl;dr: most
frames are stored as the differences between the previous one, save
the ones that have an entire image associated with them), let's take a
look at how _my streams_ in particular work. Here's a random still
frame from one of my recent streams:

<xeblog-picture path="blog/video-compression/vlcsnap-2023-02-05-11h43m29s946"></xeblog-picture>

The core elements are very simple and many of them do not change:

- There's a "facecam" window for my VTuber avatar with a background
  that changes once every minute
- There's my sigil on the left side of the frame, under my facecam
  window
- There's a cutout that the terminal window fits into (painstakingly
  positioned so that the cutout hides the Windows Terminal tab bar)

The two things that update the most often are the terminal window and
the facecam. At the time of writing this post, the overlay is static
(however I've been considering making something dynamic with Godot
that would allow me to embed the Twitch chat, but there are concerns
with privacy and user-generated content that have left me still
considering the best way to do this). This means that about 5-10
percent of the viewport _does not change_ every frame, meaning that
the bandwidth per second can be saved for the things that _do_ change.

My VTuber software VSeeFace is a bit feature-incomplete, which means
that there isn't much that changes per frame. This means that the
facecam doesn't change much either (it mostly switches between
keyframes for facsimilies of common speech sounds), which adds to even
more bitrate savings.

Consider these two random sequential frames from one of my streams:

| Frame 1 | Frame 2 |
|---------|---------|
| <xeblog-picture path="blog/video-compression/vlcsnap-2023-02-05-11h43m29s946"></xeblog-picture> | <xeblog-picture path="blog/video-compression/vlcsnap-2023-02-05-11h43m34s707"></xeblog-picture> |

Did you notice the difference? It's my VTuber avatar doing the
animation for the /s/ sound. This means that about 0.05% of the
viewport actually changed. This allows me to crank the bitrate way
down without losing out on quality as much as it would in something
like Beat Saber. Consider these two random sequential frames of me
playing [Synth Riders](https://synthridersvr.com/):

| Frame 1 | Frame 2 |
|---------|---------|
| <xeblog-picture path="blog/video-compression/vlcsnap-2023-02-05-14h59m02s427"></xeblog-picture> | <xeblog-picture path="blog/video-compression/vlcsnap-2023-02-05-14h59m07s393"></xeblog-picture> |

There is way more of a visible difference between these two frames,
and as such it requires a much higher bitrate in order to encode this
video in a way that isn't painful to watch.

## The encode-ening

As an example, let's take a look at [my most recent
stream](https://xeiaso.net/vods/2023/2/emacs-nix) where I mostly
hacked up my Nix-based Emacs config so that I could use it instead of
my Spacemacs config. In my video folder, the 1 hour, 54 minute video
file takes up about 4.82 GB of space. When I pass it to ffprobe (the
ffmpeg tool that tells you information about a media file), I get
this metadata back:

```
Input #0, matroska,webm, from 'emacs-hacking.mkv':
  Metadata:
    ENCODER         : Lavf58.76.100
  Duration: 01:53:47.14, start: 0.000000, bitrate: 6066 kb/s
  Stream #0:0: Video: h264 (High), yuv420p(tv, bt709/reserved/reserved, progressive), 1280x720 [SAR 1:1 DAR 16:9], 30 fps, 30 tbr, 1k tbn (default)
    Metadata:
      DURATION        : 01:53:47.133000000
  Stream #0:1: Audio: aac (LC), 48000 Hz, stereo, fltp (default)
    Metadata:
      title           : simple_aac
      DURATION        : 01:53:47.136000000
```

Here are the key things we can take away from this:

- It's h.264 video at 6066 kb/sec
- It's 720p video at 30 frames per second
- The audio track is
  [AAC](https://en.wikipedia.org/wiki/Advanced_Audio_Coding), which is
  the most commonly used audio codec with h.264 video files

<xeblog-conv standalone name="Cadey" mood="coffee">For the sake of
napkin-math, we're going to assume this is a two hour video, mostly
because it lets you divide the size by two for the megabytes per-hour
comparison.</xeblog-conv>

  - First attempt from the HLS announcement post
    - ffmpeg command
    - output size
    - visible quality

So, let's get a-crunching! In my [initial HLS
post](https://xeiaso.net/blog/hls-experiment) I used this command:

```
ffmpeg \
  -i ./source.mp4 \
  -level 3.0 \
  -start_number 0 \
  -hls_time 10 \
  -hls_list_size 0 \
  -f hls \
  index.m3u8
```

If I let this run I get a 470MB folder with HLS chunks in it. Each HLS
chunk is about 10 seconds of video, and on average each chunk is about
1.45MB. This gives us a total cost per hour of 235 MB. This is way
better than you'd expect from fairly uncomplicated settings, and this
is what I've used on my blog. It's not the best, not the worst, but it
gets the job done and that's good enough for most uses.

Running a random chunk through ffprobe gives me this metadata:

```
Input #0, mpegts, from 'baseline/index5.ts':
  Duration: 00:00:16.67, start: 51.421333, bitrate: 520 kb/s
  Program 1 
    Metadata:
      service_name    : Service01
      service_provider: FFmpeg
  Stream #0:0[0x100]: Video: h264 (Constrained Baseline) ([27][0][0][0] / 0x001B), yuv420p(tv, bt709/reserved/reserved, progressive), 1280x720 [SAR 1:1 DAR 16:9], 30 fps, 30 tbr, 90k tbn
  Stream #0:1[0x101]: Audio: aac (LC) ([15][0][0][0] / 0x000F), 48000 Hz, stereo, fltp, 144 kb/s
```

And it looks great:

| Frame 1 | Frame 2 |
|---------|---------|
| <xeblog-picture path="blog/video-compression/vlcsnap-2023-02-05-15h38m36s591"></xeblog-picture> | <xeblog-picture path="blog/video-compression/vlcsnap-2023-02-05-15h38m40s053"></xeblog-picture> |

However, we can go deeper. The source video was about 6000 kb/sec and
ffmpeg already crunched it down to 520 kb/sec. However, we're not even
getting into the real fun power of ffmpeg: multi-pass encoding.

### Multi-pass ~~drifting~~ encoding

The encoding I just did is known as a single-pass encode. ffmpeg
started at the beginning of the video file and encoded everything as
it got it. There was no real foreknowledge of what was going on, it
was sight-reading the video and going on the fly. A multi-pass encode
has ffmpeg scan over the entire video once and then uses that
information to encode things more optimally. In order to encode this
with multi-pass encoding, we need to have two ffmpeg commands:

```
ffmpeg \
  -i ./emacs-hacking.mkv \
  -b:v 550k \
  -level 3.0 \
  -f mp4 \
  -pass 1 \
  -y \
  /dev/null
  
ffmpeg \
  -i ./emacs-hacking.mkv \
  -b:v 550k \
  -level 3.0 \
  -start_number 0 \
  -hls_time 10 \
  -hls_list_size 0 \
  -f hls \
  -pass 2 \
  ./two-pass/index.m3u8
```

<xeblog-conv standalone name="Mara" mood="hacker">Multi-pass encoding
isn't compatible with automatically figuring out the bitrate, so the
arbitrarily chosen bitrate target of 550 kilobits per second was
chosen.</xeblog-conv>

This gets us a slightly larger result size (mostly because of the
arbitrarily chosen bitrate), but it gave a 600 MB output for 300 MB of
video per hour. This is a bit too big for what I want, but it's a good
starting point for going even deeper.

ffprobe reports this about a random HLS chunk:

```
Input #0, mpegts, from './two-pass/index277.ts':
  Duration: 00:00:08.33, start: 2776.421333, bitrate: 561 kb/s
  Program 1 
    Metadata:
      service_name    : Service01
      service_provider: FFmpeg
  Stream #0:0[0x100]: Video: h264 (Constrained Baseline) ([27][0][0][0] / 0x001B), yuv420p(tv, bt709/reserved/reserved, progressive), 1280x720 [SAR 1:1 DAR 16:9], 30 fps, 30 tbr, 90k tbn
  Stream #0:1[0x101]: Audio: aac (LC) ([15][0][0][0] / 0x000F), 48000 Hz, stereo, fltp, 131 kb/s
```

And here are two sequential frames:

| Frame 1 | Frame 2 |
|---------|---------|
| <xeblog-picture path="blog/video-compression/vlcsnap-2023-02-05-16h17m35s822"></xeblog-picture> | <xeblog-picture path="blog/video-compression/vlcsnap-2023-02-05-16h17m40s334"></xeblog-picture> |

This is passable, and I wouldn't be ashamed to have it visible on my
portfolio for my mostly text-based streams.

### We don't need no stinkin' bitrate!

In video compression, the bitrate is the amount of bits of data per
second of video. This includes keyframes. This part of the process is
where you start playing with the bitrate numbers and crunch things
down even more. In my testing, I found that 150kb/sec was the best
balance between readability of text and filesize. This means you can
use a ffmpeg command that looks like this:

```
ffmpeg \
  -i ./emacs-hacking.mkv \
  -b:v 150k \
  -profile:v high \
  -preset slow \
  -tune animation \
  -pix_fmt yuv420p \
  -f mp4 \
  -pass 1 \
  -y \
  /dev/null
  
ffmpeg \
  -i ./emacs-hacking.mkv \
  -b:v 150k \
  -profile:v high \
  -preset slow \
  -tune animation \
  -pix_fmt yuv420p \
  -start_number 0 \
  -hls_playlist_type vod \
  -hls_allow_cache 1 \
  -hls_time 0 \
  -hls_list_size 0 \
  -f hls \
  -pass 2 \
  ./two-pass-150/index.m3u8
```

There's one other major difference with how this file in particular is
encoded. I changed the `hls_time` from 10 seconds to 0 seconds. HLS is
built on chunking video into a billionty small chunks, and the
`-hls_time` flag tells ffmpeg to chunk the video at the next keyframe
after about that many seconds. This means that each HLS chunk we made
before had about 10 seconds plus the gap to the next keyframe of
video, so about 10-15 seconds on average. When we move the bitrate
down this low, we're going to need to also be more aggressive about
moving each keyframe into its own HLS chunk or things will look
slightly off in a way that people won't really be able to place their
fingers on.

<xeblog-conv name="Mara" mood="hacker" standalone>The `-tune
animation` tells ffmpeg to put more deblocking data into the resulting
files so that you notice compression artifacts less easily. Natively
each group of pixels is compressed as a superblock of a bunch of
pixels and most of the time you can't see the differences between the
pixel groups. You can see that very easily with anime and video
recordings of computer UIs, especially when the video changes a lot
very quickly. If you've ever seen things get blocky when glitter is
flying around in YouTube videos, this is why.</xeblog-conv>

The other encoding settings are something I got off of an anime piracy
forum that had suggestions for how to make anime look good at lower
filesizes. I guess you can consider my vtubing stuff to be anime if
you squint enough, so let's throw it into the pot and see what sticks.

<xeblog-conv standalone name="Cadey" mood="coffee">One of the real
annoying parts of this video conversion process is that you can't use
hardware-accelerated encoding to speed this up because most of what
we're doing here is beyond the scope of the encoding hardware in
consumer GPUs.</xeblog-conv>

After this command, I ended up with a total filesize of 267 megabytes.
This is a lot closer to what I want!

Here's the ffprobe output on a random chunk:

```
Input #0, mpegts, from './two-pass-150/index341.ts':
  Duration: 00:00:08.39, start: 2768.080167, bitrate: 258 kb/s
  Program 1 
    Metadata:
      service_name    : Service01
      service_provider: FFmpeg
  Stream #0:0[0x100]: Video: h264 (High) ([27][0][0][0] / 0x001B), yuv420p(tv, bt709/reserved/reserved, progressive), 1280x720 [SAR 1:1 DAR 16:9], 30 fps, 30 tbr, 90k tbn
  Stream #0:1[0x101]: Audio: aac (LC) ([15][0][0][0] / 0x000F), 48000 Hz, stereo, fltp, 138 kb/s
```

<xeblog-conv name="Aoi" mood="wut">Huh, the _audio_ is bigger than the
video?</xeblog-conv>

Yep! The video stream is now _so optimized_ that the audio is the
bigger concern. Here's two more sequential frames:

| Frame 1 | Frame 2 |
|---------|---------|
| <xeblog-picture path="blog/video-compression/vlcsnap-2023-02-05-16h54m33s293"></xeblog-picture> | <xeblog-picture path="blog/video-compression/vlcsnap-2023-02-05-16h54m37s397"></xeblog-picture> |

Now _this_ is how you know you've found a good balance between bitrate
and visual quality. That random HLS chunk I copied was 265 kiloBYTES
and was about 8 seconds of video. This aligns with the bitrate
estimate of 258 kb/sec that ffmpeg gave us. This is also small enough
that it's very efficient for bandwidth and for the way that XeDN's
storage architecture works. That's just about twice as much as [the
jpg form of this post's cover
image](https://cdn.xeiaso.net/file/christine-static/hero/sakura-onsen.jpg).

If you try to go with a lower bitrate, you'll start to find halos
around text characters and overall it just starts to look like YouTube
dropping to 240p when your comcast connection suddenly has a pingspike
and then it looks like if you had one dollar for every pixel in the
image, you'd have one dollar.

<xeblog-conv name="Cadey" mood="coffee">Ask me how I
know.</xeblog-conv>

### Even deeper

At this point, we have hit the practical wall for how well the video
can be encoded, but we still have a _lot_ of room in the audio
department. As of this point the video takes up about 150 kilobits per
second of data, but the audio takes up 128 kilobits per second. It
would be nice if I could use something like
[opus](https://caniuse.com/opus) for the audio and
[av1](https://caniuse.com/av1) for the video, but I can't use those
yet. I could probably get away with making multiple encodings with one
being an av1 encode and the other being a mp4 encode, but that's a bit
too complicated for my needs. Not to mention that would not help with
the space saving issue.

So, let's compress the audio. On advice of a friend, I tried to use
the `libfdk_aac` codec to get the audio down to 32 kilobits per second
but in such a way that it doesn't sound like trying to stuff a concert
symphony through the phone lines. I looked at the [documentation for
how to use `libfdk_aac`](https://trac.ffmpeg.org/wiki/Encode/AAC) and
ran into a slight snag:

> The license of libfdk_aac is not compatible with GPL, so the GPL
> does not permit distribution of binaries containing incompatible
> code when GPL-licensed code is also included. Therefore this encoder
> have been designated as "non-free", and you cannot download a
> pre-built ffmpeg that supports it. This can be resolved by compiling
> ffmpeg yourself.

I checked, and the version of ffmpeg in my package manager was not
built with `libfdk_aac`.

<xeblog-conv name="Aoi" mood="sus">Isn't compiling a custom version of
ffmpeg a huge pain needing you to track down a billion
dependencies?</xeblog-conv>
<xeblog-conv name="Mara" mood="happy">Normally, yes. But, we're using
[Nix](https://nixos.org)!</xeblog-conv>

One of the fundamental concepts of Nix is that package builds are
functions that take in an attribute set containing the build
instructions and then all that is passed to a builder function that
compiles all that metadata down to a bash script that's run in a
sandbox. Nix will handle grabbing any dependencies and putting them in
the `$PATH` of the build environment. It all Just Works™️.

There are two ways you can
[override](https://ryantm.github.io/nixpkgs/using/overrides/) how
package builds work:

- Changing the arguments passed into the package's function
- Changing the attributes passed into the builder function

In this case, we need to change the configuration flags and build
dependencies of ffmpeg, so we're going to opt for overriding the
attributes of the build with the `.overrideAttrs` method that every
package has available. Here's the override we need:

```nix
pkgs.ffmpeg_5-full.overrideAttrs (old: rec {
    configureFlags = old.configureFlags ++ [ "--enable-libfdk_aac" "--enable-nonfree" ];
    buildInputs = old.buildInputs ++ [ pkgs.fdk_aac ];
})
```

This overrides the `buildInputs` (native dependencies, for things like
openssl and command-line tools) and the `configureFlags` (the
arguments passed to `./configure`, the entry point for most C/C++
build systems) to add support for `libfdk_aac`. We can then slap that
in a `flake.nix` file and make a `devShell` with that prepopulated in
the environment:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixpkgs-unstable";
    utils.url = "github:numtide/flake-utils";
  };

  outputs = { self, nixpkgs, utils }:
    utils.lib.eachDefaultSystem (system:
      let
        pkgs = import nixpkgs {
          inherit system;
          overlays = [ ];
        };

        ffmpeg = pkgs.ffmpeg_5-full.overrideAttrs (old: rec {
          configureFlags = old.configureFlags ++ [ "--enable-libfdk_aac" "--enable-nonfree" ];
          buildInputs = old.buildInputs ++ [ pkgs.fdk_aac ];
        });
      in rec { devShell = with pkgs; mkShell { buildInputs = [ ffmpeg ]; }; });
}
```

<xeblog-conv standalone name="Mara" mood="hacker">If you want an
exercise to help you get more up to speed with Nix flake hacking, try
changing this `flake.nix` file so that it exposes the custom ffmpeg
build as a package.</xeblog-conv>

Now the ffmpeg commands can look like this:

```
ffmpeg \
  -i ./emacs-hacking.mkv \
  -b:v 150k \
  -profile:v high \
  -preset slow \
  -tune animation \
  -pix_fmt yuv420p \
  -f mp4 \
  -pass 1 \
  -an \
  -y \
  /dev/null
  
ffmpeg \
  -i ./emacs-hacking.mkv \
  -b:v 150k \
  -profile:v high \
  -preset slow \
  -tune animation \
  -pix_fmt yuv420p \
  -start_number 0 \
  -hls_playlist_type vod \
  -hls_allow_cache 1 \
  -hls_time 0 \
  -hls_list_size 0 \
  -f hls \
  -c:a libfdk_aac \
  -profile:a aac_he_v2 \
  -b:a 32k \
  -pass 2 \
  ./two-pass-fdk/index.m3u8
```

This will let us end up with a 178 megabyte folder. ffprobe will let
us confirm that we've won:

```
Input #0, mpegts, from './two-pass-fdk/index341.ts':
  Duration: 00:00:08.39, start: 2768.163000, bitrate: 146 kb/s
  Program 1 
    Metadata:
      service_name    : Service01
      service_provider: FFmpeg
  Stream #0:0[0x100]: Video: h264 (High) ([27][0][0][0] / 0x001B), yuv420p(tv, bt709/reserved/reserved, progressive), 1280x720 [SAR 1:1 DAR 16:9], 30 fps, 30 tbr, 90k tbn
  Stream #0:1[0x101]: Audio: aac (HE-AACv2) ([15][0][0][0] / 0x000F), 48000 Hz, stereo, fltp, 33 kb/s
```

This is probably the floor for getting reasonably-sounding audio. The
end result is 89 megabytes of data per hour of stream footage. It
should also be _maximally compatible_ with devices and allow me to
save money on bandwidth and storage. The chunks are about the size of
1-1.5 JPEG images too, which means that they are very efficient for
XeDN's storage/retrieval architecture.

## Caveats

The only problem is that it's very optimized for my stream VODs in
particular and if I wanted to encode Synth Riders gameplay I'd need to
vastly bump up the video and audio quality or it would look
_terrible_. As an example, here's the Synth Riders gameplay that I
recorded for the earlier frame comparisons:

<xeblog-video path="blog/video-compression/synth-riders"></xeblog-video>

<xeblog-conv name="Cadey" mood="coffee">The note approach speed in
that Synth Riders clip may look a bit fast at first glance, but it's
actually a bit slow for me. It's a lot easier to get into flow with a
faster approach speed, if only because it makes reading upcoming notes
easier.</xeblog-conv>

This allows me to crunch a 248 MB file down to 68 MB. I had to
compromise on the audio quality because the AAC codec profile I used
was tuned for human speech, an understandable thing to want for my
streams of mostly human speech. For that video in particular, I used
these flags:

```
-c:a libfdk_aac -b:a 64k
```

This tells ffmpeg to use `libfdk_aac` as the audio codec and sets a
bitrate of 64 kilobits per second. I don't specify a codec profile
here because I want to get the low effciency codec, which is the
default. This stack of codecs gets you roughly the same audio
experience that you get from Spotify or Apple Music (unless you enable
lossless or Dolby Atmos audio), so it should be good enough.

If I didn't bother to optimize the compression, I'd end up with a 9 MB
video that looks and sounds like this:

<xeblog-video path="blog/video-compression/synth-riders-vtuber-quality"></xeblog-video>

This is a bit interesting because before I did the compression, I
didn't think the audio would turn out as well as it did. You can
really hear the high ends of the song get absolutely destroyed (the
codec profile I use for VTubing videos is optimized for human speech,
and human speech doesn't have many high end sounds), especially when
compared to the previous recording.

---

Overall I'm quite happy with the results of this. I have an offroad
from YouTube and I can host all the video on my own infrastructure.
For most users there's no difference between this solution and
something hosted on YouTube (save the lack of a view count, comments,
and other anti-features like like/dislike buttons). I'm pretty sure
that this will meet my needs and allow me to self-host the things I
create.
