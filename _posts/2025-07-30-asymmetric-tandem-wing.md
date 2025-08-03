---
title: Failing to fly the Asymmetric Tandem Wing
date: 2025-07-30 17:34:43 +0100
categories: [Other]
tags: [uas, uav, ardupilot, arduplane, foamboard]     # TAG names should always be lowercase
description: Attempting to make the asymmetric tandem wing fly
toc: false
media_subpath: /asym/
image:
  path: DSC02773.jpg
  alt: ASW28 ready for flight
---

I've been splitting my time between a few projects in the last couple of months, all with low to middling success, so it's about time to document one of those that didn't really go to plan. I felt like exploring another swing wing configuration after the success of my oblique wing designs, and I settled on an asymmetric tandem wing planform. This planform has a fore and aft wing, but each wing only extends from one side of the fuselage, in this case a front left and rear right wing. For a practical application, the niche for this configuration being beneficial is very slim. 

The addition of each wing being articulating allows for the overall package to be small when the wings are stowed, so this configuration could be used for a tube-launched vehicle with extending winds. For a fixed fuselage length, since the wing hinges are at the forward and aft extremities of the fuselage, the asymmetric tandem wing can support a longer wing length than alternative configurations which typically hinge somewhere in the middle of the fuselage for aerodynamic balance reasons. The tandem wing ensures that the aircraft will be flyable for a CG marginally forward of the midpoint between the two wings.

## Version 1

I initially designed a small sub-250 g droppable glider with fixed wings to test the stability of the concept. A simple set of 3D printed wings and full span elevons attached to a carbon boom formed the airframe, and I included a small flight controller to provide stability augmentation. I'd wanted to try one of these cheap no-name F405 boards for some timne and this seemed the perfect opportunity to do so. The board, a FlyingRC F4Wing Mini Mk1, supports iNav and ArduPilot firmwares, and I flashed 4.6.0 ArduPlane compiled for the compatible MatekF405-TE target. The install worked flawlessly and I set up the servos with the correct elevon responses - counterintuitively setting both outputs as right elevons due to the front left wing acting as a canard (actually a canard plus aileron... perhaps a canarderon?). Having no motor output means no other setup is really needed, with my spare transmitter bound and the flight modes set to FBWA and manual.

Recently I had put together a cheap quad using a knockoff F450 frame and some spare parts given to me by a flying club member, along with an old flight controller with a broken servo rail BEC. I installed a payload release mechanism and some longer leg attachments and went out to test fly.

![Version 1](20250704_122840.jpg)
![Quad 1](DSC02782.jpg)
![Quad 2](DSC02783.jpg)

Although I had expected some loss in performance from adding the glider as a payload, the effect was more serious than I thought. Several flights were unsuccessful as the quad couldn't leave ground effect, or would lose lift during climbout. Initiaslly I thought this could be due to the glider wings blocking the quad prop downstream streamtube, so I hung the glider from a piece of string. This also didn't help much, and particularly on a windy day the weight caused some issues with control of both the quad and glider. The few successful drops ended in no control of the glider, with several CG positions tried. Ultimately, I decided to abandon the quad release idea as I wanted to further develop the design and the quad clearly didn't have the lifting capacity for a larger payload.

## Version 2

Despite not proving out the viability using the small glider, I pushed ahead with a larger powered prototype. After building my second version of the oblique wing earlier this year, I wanted to try something other than a fully 3D printed model, so I returned to foamboard as the material for the wings, with a carbon boom similar to the glider. It's been a long while since working with foamboard, so the first attempt at building was very poor. On the second attempt, I found the paper face skin on the Hobbycraft foamboard was attached with a very water-soluble glue, so a small amount of dampening caused the face to be peeled off relatvely easiely. Removing the inner paper face and scoring the foam at the points of maximum curvature made forming the foamboard to the ribs straightforward. The ribs were simple 3D printed parts from a NACA2421 airfoil, chosen to minimise the amount of sharp curvature for foamboard forming. The all up weight was about 900 g, with a 3S battery and motor from my original ASW28 model.

![Version 2](DSC02786.jpg)
![Version 2](DSC02787.jpg)

The first flight attempts with 3S were not very promising, and with the weather closing in I rushed to attempt the maiden flight. This first flight 