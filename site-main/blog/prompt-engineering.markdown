---
title: "Prompt engineering is hard"
date: 2022-10-01
tags:
 - stablediffusion
 - ai
 - madewithai
---

<xeblog-hero ai="Waifu Diffusion v1.2" file="catgirl-fireworks" prompt="girl with long green hair with cat ears wearing an orange kimono, nighttime, many fireworks, festival, happy happy, A beautiful landscape, studio ghibli, zen gates, yin yang, pagoda, colorful sky, by hayao miyazaki, starbucks vibes"></xeblog-hero>

I've seen a lot of comments on Twitter that seem to completely misunderstand the
process of getting a decent result with AI generators like Stable Diffusion and
DALL-E 2. People seem to assume that it's just "push button, recieve bacon"
without any real creativity in the equation. As someone who has done a lot of
this experimentation in the past few months, I'd like to challenge that
assertion and show you what the process for getting a decent result actually
involves.

First, you need to start off with a vision for what you want. I'm going to pull
my fictional world Malto, specifically an area named Kanar. It is a very green
area, lots of bamboo and the local architecture takes advantage of it. The area
is fairly wealthy because they take advantage of their weird soil composition in
order to produce the plants that help them make an alcoholic beverage that the
nobles all over the world can't get enough of.

From here, I like to get the "vibe" of the image down first. I think that [Waifu
Diffusion](https://github.com/kaitas/waifu-diffusion) would be a good model to
use for this, mostly because you can feed it danbooru style tags to prompt the
image you want. I also kind of want a studio ghibli feeling, and Waifu Diffusion
has proven to be very good at that.

My first iterations start out with a few 512x512 images with a few vibe prompt
keywords to get basic thoughts onto the "canvas".

Here is my starting prompt:

> `bamboo bamboo_forest grass studio_ghibli hayao_miyazaki happy peaceful summer`

| <xeblog-picture path="blog/prompt-engineering/vibes/seed_447543_00000"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/vibes/seed_447544_00001"></xeblog-picture> |
|:--- |:--- |
| <xeblog-picture path="blog/prompt-engineering/vibes/seed_447545_00002"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/vibes/seed_447546_00003"></xeblog-picture> |

<xeblog-conv name="Mara" mood="happy">Click on any of these images to open a
larger version of it!</xeblog-conv>

I'm not a big fan of these. I want a _landscape_, but it instead showed me a
bamboo forest from the inside. There were also some subjects in frame in a few
of those too. We're not focusing on subjects yet. Let's remove the
`bamboo_forest` tag and add the `landscape` and `pagoda` tags:

> `bamboo landscape pagoda grass studio_ghibli hayao_miyazaki happy peaceful summer`

| <xeblog-picture path="blog/prompt-engineering/vibes2/seed_320353_00000"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/vibes2/seed_320354_00001"></xeblog-picture> |
|:--- |:--- |
| <xeblog-picture path="blog/prompt-engineering/vibes2/seed_320355_00002"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/vibes2/seed_320356_00003"></xeblog-picture> |

This is a lot closer to what I want. I'm going to stick with this seed too,
`320353`. Now that we have a seed that's better, let's increase the resolution
to 1280x512 to see how that changes things. The AI natively draws images in
512x512 chunks, so the jump to something larger can be weird at times.

| <xeblog-picture path="blog/prompt-engineering/vibes2-wide/seed_320353_00000"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/vibes2-wide/seed_320354_00001"></xeblog-picture> |
|:--- |:--- |
| <xeblog-picture path="blog/prompt-engineering/vibes2-wide/seed_320355_00002"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/vibes2-wide/seed_320356_00003"></xeblog-picture> |

That came out way better than expected! Usually doing the jump to 1280x512
causes some cursed results and mutant hell creatures. This is actually somewhat
decent, but I want something a bit more stylized. This is roughly correlating to
the image I have of Kanar in my head, but this looks closer to a drawing of the
area done by an outsider rather than how they would portray themselves. I'm
going to add a few style keywords:

> `bamboo landscape pagoda grass studio_ghibli hayao_miyazaki happy peaceful summer ukiyo-e wood-block`

| <xeblog-picture path="blog/prompt-engineering/ukiyo-e/seed_320353_00000"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/ukiyo-e/seed_320354_00001"></xeblog-picture> |
|:--- |:--- |
| <xeblog-picture path="blog/prompt-engineering/ukiyo-e/seed_320355_00002"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/ukiyo-e/seed_320356_00003"></xeblog-picture> |

Something fun you can do is add "masterpiece" to make the image look better and
"unreal engine" to improve the lighting. Let's mess with that here:

> `bamboo landscape pagoda grass studio_ghibli hayao_miyazaki happy peaceful summer ukiyo-e wood-block unreal engine masterpiece very beautiful`

| <xeblog-picture path="blog/prompt-engineering/unreal-engine/seed_320353_00000"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/unreal-engine/seed_320354_00001"></xeblog-picture> |
|:--- |:--- |
| <xeblog-picture path="blog/prompt-engineering/unreal-engine/seed_320355_00002"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/unreal-engine/seed_320356_00003"></xeblog-picture> |

You know what, I'm not feeling that wood-block style. Let's remove it in the
next round. The main thing we need to focus on next is the subject. The main
export of Kanar is a rice-based alcoholic drink. I'm going to add "rice_paddy"
to the prompt just after `grass`:

> `bamboo landscape pagoda grass rice_paddy studio_ghibli hayao_miyazaki happy peaceful summer ukiyo-e unreal engine masterpiece very beautiful`

| <xeblog-picture path="blog/prompt-engineering/rice-paddy/seed_320353_00000"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/rice-paddy/seed_320354_00001"></xeblog-picture> |
|:--- |:--- |
| <xeblog-picture path="blog/prompt-engineering/rice-paddy/seed_320355_00002"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/rice-paddy/seed_320356_00003"></xeblog-picture> |

I think that last one is going to be the image that I'm going with. Let's see
what happens if we change the time of day:


| <xeblog-picture path="blog/prompt-engineering/times/morning"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/times/afternoon"></xeblog-picture> |
|:--- |:--- |
| <xeblog-picture path="blog/prompt-engineering/times/evening"></xeblog-picture> | <xeblog-picture path="blog/prompt-engineering/times/nighttime"></xeblog-picture> |

...well that didn't change the time of day at all. I think I like the nighttime
result though, so I'm going to go with that. Here's the image we have so far:

<xeblog-picture path="blog/prompt-engineering/times/nighttime"></xeblog-picture>

This has a bit of an imposing feeling, maybe like a castle or some other
important building. Maybe this is the private pagoda of their leader. What if we
add some guards?

> `nighttime bamboo landscape pagoda grass rice_paddy studio_ghibli hayao_miyazaki happy peaceful summer ukiyo-e unreal engine masterpiece very beautiful guards pikemen`

<xeblog-picture path="blog/prompt-engineering/rice-paddy-guards"></xeblog-picture>

I really like this. This is the kind of vibe that I'm going for. I want
something that makes me _feel_ like I'm looking into that area that I've only
ever seen in my mind's eye from description paragraphs and topological charts.
This is the kind of thing that Stable Diffusion and similar models let you do as
a writer: they let you bring images out of your head and onto the canvas so that
you can have people really understand what it's like. If I wrote a longer story
set in here, I'd probably throw this image and a few others generated with
different seeds to an artist to help me make an image for a book cover.

I'm also not really sure why people call this "prompt engineering", I'd
personally rather call it "scrying", but I can understand why Silicon Valley
culture would push everything towards being "engineering". I just legally can't
call myself an "engineer" in Canada without an engineering degree.

This is the kind of technology I am really excited for, and I can't wait to see
how this evolves. Computers are fun sometimes.
