---
title: "Site Update: HLS support"
date: 2022-10-11
tags:
 - hls
---

Hi blog readers! It's time for a regular update on stuff I've ~~gained expertise
in~~ frantically googled and hacked together so I can put it in front of your
faces.

I use YouTube as a video hosting service because it's largely predictable and
it's the evil I know. I'm starting to try to reduce my dependencies on large
centralized systems like YouTube and one of the ways I want to do this is by
hosting my own video.

In a perfect world, I'd wait until [AV1](https://en.wikipedia.org/wiki/AV1) is
fully dominant and [supported on all the browsers that you people
use](https://caniuse.com/av1), but I want to start experimenting on this now. As
such, I am using [hls.js](https://github.com/video-dev/hls.js) to allow me to
use [HTTP live streaming](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) to
embed video on my blog. You can see it on these talk pages:

- [My Blog is Hilariously Overengineered to the Point People Think it's a Static
  Site](https://xeiaso.net/talks/how-my-website-works)
- [How Static Code Analysis Prevents You From Waking Up at 3AM With Production
  on Fire](https://xeiaso.net/talks/conf42-static-analysis)

And if you want a live demo, check it out here with footage of a random Splatoon
3 match that I've been using to benchmark various ffmpeg flags:

<noscript><xeblog-conv name="Mara" mood="hacker">NoScript users, you will have
to enable JavaScript for this one, sorry!.</xeblog-conv></noscript>

<xeblog-video path="blog/hls-post"></xeblog-video>

As far as I know, the only issue with this is that it doesn't work on iOS
Safari. I'm not sure why. I plan to get this fixed at some point, but for now
this works enough to experiment with. I will be sure to upload my next talk
video for my GoLab talk using this.

Here is the ffmpeg command I'm using if it helps you figure out what I'm doing
wrong:

```
ffmpeg \
  -i ./source.mp4 \
  -profile:v baseline \
  -level 3.0 \
  -start_number 0 \
  -hls_time 10 \
  -hls_list_size 0 \
  -f hls \
  index.m3u8
```

I also have a Pixel 6A running GrapheneOS now and I plan to experiment with
GrapheneOS as my next daily driver smartphone OS. I enjoy how spartan it is.
Just look at the default homescreen:

<xeblog-picture path="blog/grapheneos_home_screen"></xeblog-picture>

It's beautiful. There's just _nothing_ there. I love it so much. I'm working on
a longer post where I write about my experiences with GrapheneOS and my thoughts
on if I want to use it as a daily driver. It reminds me of simpler days where
people competed to build ever more slimmed down Android roms for budget phones.

Moving off iOS is going to be very multifaceted and complicated.

Anyways, hope this is interesting! Be well.
