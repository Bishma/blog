---
title: "Just A Super [glue] Organizer"
date: 2021-01-23T18:36:03-08:00
description: A super glue organizer designed for a prusaprinters.org contest.
images:
- /images/CAddy/tray front - full.jpg
draft: false
tags: [ "fusion360", "design", "organization", "3d-printing" ]
---

# The CAddy

When you do a lot of 3D printing you use a lot of super glue (AKA CyanoAcrylate, AKA "CA"). It's a crack fixer, gap filler, layer re-adherer, [classic movie line](https://www.youtube.com/watch?v=hd1ciPnTGKg), and I think you can use it to stick things together or something. Moreover different use cases require different formulations, and the bottle you can find is never the one you need (I think it says so on the label).

So I decided to design a carrier for all my CA. A "CAddy," if you will. And because that sounded kind of boring I decided to design it around a super glue molecule (specifically [ethyl-2-cyanoacrylate](https://en.wikipedia.org/wiki/Ethyl_cyanoacrylate))

{{< layout/image-squash >}}

{{< layout/image-squash-element gallery="CAddy Gallery" image="/images/CAddy/tray front - full.jpg" caption="Front view of the CAddy tray." >}}

{{< layout/image-squash-element gallery="CAddy Gallery" image="/images/CAddy/tray back - full.jpg" caption="Rear view of the CAddy tray." >}}

{{< layout/image-squash-element gallery="CAddy Gallery" image="/images/CAddy/tray front - empty.jpg" caption="Front view of the CAddy tray while empty." >}}

{{< layout/image-squash-element gallery="CAddy Gallery" image="/images/CAddy/tray top - empty.jpg" caption="Top view of the CAddy tray while empty." >}}

{{< layout/image-squash-element gallery="CAddy Gallery" image="/images/CAddy/on buildplate.jpg" caption="The tray as printed." >}}

{{< layout/image-squash-element gallery="CAddy Gallery" image="/images/CAddy/knife handle on build plate.jpg" caption="The utility knife handle as printed." >}}

{{< /layout/image-squash >}}

I had this idea a while ago because I was sick of my glue bottles getting separated. The thing that finally got me to finish it up is a [contest on prusaprinters.org](https://blog.prusaprinters.org/organization-contest-announcement_42886/).

## Features

Because it's not a Derek project if its not over-engineered I've packed as many features as I could make fit.

* Slots for up to 4 bottles with {{< lightbox-link gallery="features" image="/images/CAddy/models.png" caption="Fusion 360 rendering showing the tray's front view, with and without text labels." >}}optional labels{{< /lightbox-link >}}
  * Sized for 50g bottles of [Hobby King glues](https://hobbyking.com/en_us/hobbyking-super-glue-ca-50g-1-7oz-super-thin.html)
  * Specifically, the ovals that are {{< lightbox-link gallery="features" image="/images/CAddy/glue hole size.png" caption="Fusion 360 sketch showing the oval dimensions: 50mm x 30mm" >}}50mm on the long side and 30mm on the short side{{< /lightbox-link >}}
* The oxygen at the 4 position will hold an acupuncture needle (useful for clog clearing)
* The triple bond holds a {{< lightbox-link gallery="features" image="/images/CAddy/knife assembly.jpg" caption="Assembly of the utility knife." >}}utility knife{{< /lightbox-link >}}
  * The printed handle holds a #1 utility blade screws together for safety
* The other oxygen atom is big enough for a sharpie or a small rolled up rag / sandpaper
* The front has a small slot sized to hold a few craft (popsicle) sticks
* The end of the molecule is a textured handle

## Design

I had two starting points. First I wanted it design it around the CA molecular line diagram. I easily found the diagram in [SVG format](https://en.wikipedia.org/wiki/Ethyl_cyanoacrylate#/media/File:Ethyl_cyanoacrylate.svg) on wikipedia. I aligned the bottle cutouts so they point at the carbon junctions and was pleased with the results.

Secondly, I was able to find a step model of a #1 utility knife on
[grabcad](https://grabcad.com/library/utility-blade-3), size a handle around it, and perform a [boolean subtraction](https://knowledge.autodesk.com/support/fusion-360/getting-started/caas/screencast/Main/Details/1365f377-8bf4-4845-92af-4b775858a9d6.html) and have a nice starting point for the utility knife handle. Adding tolerances for FDM was a bit of a challenge and results in the blade having a little bit of play in the handle. But as long as downward pressure is being applied there is a machine screw reenforcing the main stress point. I don't think different dimensional accuracies will affect function or safety. Use your best judgement.

## Printing

### Settings and Assembly

#### Tray

* 0.2mm layer height
* 0.4mm nozzle
* No supports, rafts, or brims
* 10% infill
* The color change is done via filament swap for the final 6 layers.

#### Knife 

* 0.15mm layer hight
* 0.4 mm nozzle
* No supports, rafts, or brims
* 15% infill

The knife stores in the CAddy body blade side down with a backstop to prevent the tip of the blade from digging into the bottom of the tray. Please don't cut yourself.

The filaments pictured are Prusament PLA in Galaxy Purple and Printed Solid Jessie in Feta Green.

### Availability

* {{< model-link 53618 >}}
* [Github](https://github.com/Bishma/admafu/tree/main/CAddy)