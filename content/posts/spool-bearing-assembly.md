---
title: "Spool Bearing Assembly"
date: 2020-11-25T16:08:04-08:00
draft: false
tags: [ "3d-printing", "design", "assembly", "overengineering" ]
---

# Filament Spool Bearing Assembly for a 3/4" dowel

One of the issues I encountered when I decided to try to make a filament dry box out of material I had on hand was that my rod (a 19mm dowel wooden left over from Halloween) was causing too much friction. I knew a heavy spool on a narrow rod would slide rather than roll, but I didn't anticipate just how much force would be required to feed filament.

I did a little research and figured out that a bushing would solve my issue (basically adapting the 19mm rod to make it a 50mm rod)... but I also have a ton of crappy 608 bearings that I got cheap and, you know, bearings are more fun than bushings. {{< shrug >}}

I figured three points of contact would be sufficient for the weight but three 608 bearing + the diameter of the rod is greater than the inner diameter of a filament spool. That means that the end caps are as thick as a 608 bearing + the plastic to hold them in place. I pressed ahead anyway knowing this means I can only fit 3 spools in my dry box instead of 4. I used brass inserts for screwing that parts of the caps together. I did so mainly because I has a new soldering iron tip I wanted to try. Should I need more caps in the future I'll probably tweak the design to use M3 hex nuts.

{{< layout/image-squash >}}

{{< layout/image-squash-element gallery="Assembly Parts" image="/images/gallery/bearing-assembly/1_cap-parts.jpg" caption="The parts that make up the end caps." >}}

{{< layout/image-squash-element gallery="Assembly Parts" image="/images/gallery/bearing-assembly/2_wide-spindle-assembled.jpg" caption="Fully assembled with the long version of the spindle." >}}

{{< layout/image-squash-element gallery="Assembly Parts" image="/images/gallery/bearing-assembly/3_assembly-caps.jpg" caption="Assembled end caps." >}}

{{< layout/image-squash-element gallery="Assembly Parts" image="/images/gallery/bearing-assembly/4_in-use.jpg" caption="In use, holding filament spools in my dry box." >}}

{{< layout/image-squash-element gallery="Assembly Parts" image="/images/gallery/drybox/3_filament-guides.jpg" caption="Front view of my dry box with the bearing assemblies in use." >}}

{{< /layout/image-squash >}}

## Design

This was pretty straight forward. I looked up the dimensions of a 608 bearing and the average diameter for a 1kg filament spool's hole. In Fusion 360 I used the radius of the bearing and the diameter of the dowel to layout the caps. My first prototype technically worked but I forgot to add any tolerance and so there was no play between the assembly and the dowel.

{{< youtube 2FpKSwWCUTk >}}

It was still better than it was without anything, but I was sure the  weight of the spools and the tightness of the bearings would result in grooves or flat spots on my wooden dowel. So I moved the bearings 0.15mm further from the rod. Now, if anything, it might spin too well. When the printer feeds the spools inertia can cause it to keep rolling a bit allowing some slack to build up.

{{< youtube heie4OLFuSw >}}

I'm pretty happy with how the whole dry box project turned out. I'll have a post about it soon.