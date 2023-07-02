---
title: Compiling Code to Matter in My Living Room
date: 2022-03-28
tags:
 - openscad
 - 3dprinting
---

In a moment of weakness, my husband and I got a 3d printer. It's mostly been sitting around and not doing much since we got it, but recently I found a great use for it: I wanted a controller stand for my Valve Index controllers and VR full body trackers.

After doing some digging on Thingiverse, I found [this stand](https://www.thingiverse.com/thing:4587097) that looked like it had promise. So I downloaded the model, sliced it and then sent it over to Kyubey:

<blockquote class="twitter-tweet"><p lang="tl" dir="ltr">Kyuubey is happy <a href="https://t.co/atTLN8MSgc">pic.twitter.com/atTLN8MSgc</a></p>&mdash; Xe Iaso (@theprincessxena) <a href="https://twitter.com/theprincessxena/status/1507485129907871747?ref_src=twsrc%5Etfw">March 25, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

[Kyubey's name is a reference to <a href="https://madoka.fandom.com/wiki/Kyubey">Kyubey</a> from Puella Magi Madoka Magika</a>.](conversation://Mara/hacker)

Once it was done I ended up with a stand that I could feed [these cables I got from Amazon](https://www.amazon.ca/gp/product/B09LSF8XL9/) through. The tracker holes worked great, but the controller holes were just barely too small.

This was kinda frustrating and I almost gave up on the project, but then I remembered that [OpenSCAD](https://openscad.org) existed. OpenSCAD is a weird programming environment / 3D modeling hybrid program that I've seen used on Thingiverse. It works by letting you position platonic solids into a 3d environment, and from there you can create anything you want.

One of the primitives that OpenSCAD offers is a cylinder. So I wondered if I could use one of those to widen the hole in the index stand and then reprint the part with the wider hole.

[Wait, you're using a CAD program to fix your 3D print by modifying the model instead of using, I don't know, a drill and 5 minutes to make it fit that way?](conversation://Numa/dismay)

[There's no doing like overdoing!](conversation://Cadey/enby)

After some finangling, I managed to get the cylinders in the right place with this OpenSCAD code:

```scad
//difference() {
    color("magenta") translate([0, 0, 0]) import("./assets/ValveTrackerDeckEditedByInugoro.stl");
    // bores for controller holders
    color([0, 1, 0]) translate([63, 44, 0]) cylinder(h = 55, r = 4.75);
    color([0, 1, 0]) translate([-63, 44, 0]) cylinder(h = 55, r = 4.75);
//}
```

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Some finagling required <a href="https://t.co/7T0R6x1XoP">pic.twitter.com/7T0R6x1XoP</a></p>&mdash; Xe Iaso (@theprincessxena) <a href="https://twitter.com/theprincessxena/status/1508566854926745614?ref_src=twsrc%5Etfw">March 28, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

And when I uncommented out the `difference()` block, it ends up looking good enough:

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="und" dir="ltr"><a href="https://t.co/fiShvlN8QH">pic.twitter.com/fiShvlN8QH</a></p>&mdash; Xe Iaso (@theprincessxena) <a href="https://twitter.com/theprincessxena/status/1508567556759728141?ref_src=twsrc%5Etfw">March 28, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

So then I took a good solid look at the rest of the 3D printed part to see if I could improve on anything else before I sent it to another round of the printer. The last stand took _14 hours_ to print and used a lot of material. I want to avoid waste.

Something I noticed is that the front of the print where all the cables come out was a bit too thin. All 5 of the cables wouldn't fit in there (my braided cables must have been thicker than the ones that the original modeler used). So again I grabbed a few platonic solids and managed to make it work out:

```scad
// widen the paths
color("green") translate([0, -16, 1.3]) rotate([0, 0, 90]) cube([10, 57, 7.8], center = true);
color("green") translate([0, 0, 1.7]) rotate([0, 0, 0]) cube([25, 30, 7], center = true);
```

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="und" dir="ltr"><a href="https://t.co/pKAVtiPfDS">pic.twitter.com/pKAVtiPfDS</a></p>&mdash; Xe Iaso (@theprincessxena) <a href="https://twitter.com/theprincessxena/status/1508568858650685440?ref_src=twsrc%5Etfw">March 28, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Then I wanted to add some wedges into the underside of the part to help me get the print off the bed. Most people have a problem with bed adhesion being too little. I have too much bed adhesion. So I added some angled rectangles:

```scad
// wedges to help get the print off the bed
color([1, 1, 0]) translate([-120, 0, 0]) rotate([15, 0, 90]) cube([10, 11, 2], center = true); // right
color([1, 1, 0]) translate([120, 0, 0]) rotate([-15, 0, 90]) cube([10, 11, 2], center = true); // left
color([1, 1, 0]) translate([0, -85, 0]) rotate([0, 15, 90]) cube([10, 11, 2], center = true); // back
color([1, 1, 0]) translate([60, 56, 1]) rotate([0, -15, 90]) cube([10, 11, 2], center = true); // front left
color([1, 1, 0]) translate([-60, 56, 1]) rotate([0, -15, 90]) cube([10, 11, 2], center = true); // front right
color([1, 1, 0]) translate([32.5, 41, 1]) rotate([0, -15, 130]) cube([10, 11, 2], center = true); // front left inner
color([1, 1, 0]) translate([-32.5, 41, 1]) rotate([0, -15, 60]) cube([10, 11, 2], center = true); // front right inner
```

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="und" dir="ltr"><a href="https://t.co/XUQ9ZeYk1H">pic.twitter.com/XUQ9ZeYk1H</a></p>&mdash; Xe Iaso (@theprincessxena) <a href="https://twitter.com/theprincessxena/status/1508569796253827077?ref_src=twsrc%5Etfw">March 28, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

And then once I spun it around for a bit and thought it was good, I sliced it in PrusaSlicer and sent it off to Kyubey. It was going to take 14 hours, so I went off to do other things, ate dinner and then went to bed while the printer continued.

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="fr" dir="ltr">Diligent bean <a href="https://t.co/yPgnJA0ZdW">pic.twitter.com/yPgnJA0ZdW</a></p>&mdash; Xe Iaso (@theprincessxena) <a href="https://twitter.com/theprincessxena/status/1508397506031460352?ref_src=twsrc%5Etfw">March 28, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Then when I woke up, Kyubey was done:

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="und" dir="ltr"><a href="https://t.co/2E1IS810EH">pic.twitter.com/2E1IS810EH</a></p>&mdash; Xe Iaso (@theprincessxena) <a href="https://twitter.com/theprincessxena/status/1508407046995156992?ref_src=twsrc%5Etfw">March 28, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

I was excited and chiseled the print off the bed (the wedges helped a little, but it ended up making the print look kinda weird so I don't know if I will do that again), but the hole for the middle tracker didn't fit perfectly. Everything else did though.

[If you want to get prints off your printer easier, see this video for the method we're starting to use: <br /><br /><iframe width="560" height="315" src="https://www.youtube.com/embed/VCCbzCvtRzU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>](conversation://Mara/hacker)

I looked on my desk and found that a random pen that I had sitting around for months was about the right size, so I pushed it into and out of the hole a few times and then the cables fit perfectly. I assume some plastic was in a weird state or something.

Then I set everything up and I had my Index controller stand:

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">Victory! <a href="https://t.co/A3aCtQMQt5">pic.twitter.com/A3aCtQMQt5</a></p>&mdash; Xe Iaso (@theprincessxena) <a href="https://twitter.com/theprincessxena/status/1508426229464064001?ref_src=twsrc%5Etfw">March 28, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

[I really need to get a table or something for this.](conversation://Cadey/facepalm)

I've uploaded my modified version to [Thingiverse](https://www.thingiverse.com/thing:5332988). If you want to see the OpenSCAD code, you can check it out on GitHub [here](https://github.com/Xe/3dstuff/blob/main/index_stand_hack.scad). I'm really liking OpenSCAD so far. It's very weird but it lets you do whatever you want by chaining together basic shapes to build up to what you want. I imagine I will be using it a lot in the future, especially once my husband's new sim racing gear comes in.

Having a 3D printer around is like having a very weird superpower on standby. You can compile matter in your living room, but you need a very pedantic description of what that should look like. You also can have any material you like as long as it's plastic. However when it's useful, it's a lifesaver. You can make something to fit a gap or mend something broken or even add functionality to something that lacked it. The cloud's the limit!
