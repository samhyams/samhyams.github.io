---
title: Thermal Prep Time
date: 2025-05-13 18:15:23 +0000
categories: [Projects, ASW28]
tags: [uas, uav, ardupilot, arduplane]     # TAG names should always be lowercase
description: Building a new ASW28 ready for summer soaring
toc: false
media_subpath: /asw28/
image:
  path: DSC02773.jpg
  alt: ASW28 ready for flight
---

It's been a few years since my original Volantex ASW28 met its demise on the slopes at Bratton Camp above the Westbury white horse, but the intervening summers have passed with growing regret of not replacing that aircraft with something similarly adept at catching a thermal on a warm day. Not wanting to miss another summer, I hastily bought a set of components to replicate and improve on the original, and settled on a new copy of the same ASW28 airframe, not finding another model that could compete as a value proposition.

## Building

This iteration of the ASW28 has a couple of minor, but key, improvements. I've ditched Matek for the flight controller and have tried Holybro's H743 Wing for the first time. It's a neatly laid out flight controller and is mostly in line with what you might expect from a H7 wing controller on the market, with two minor quirks. The default servo rail voltage is 6 V instead of 5 V which is more commonplace. Most flight controller PDBs will have the option to uprate from 5 to 6 V using a solder bridge, but Holybro have instead opted to default to 6 V with the option to uprate to 8 V. This isn't an issue as most servos will happily run on 6 V with improved response - even the cheap servos included in the Volantex kit - but is important to keep in mind if you are planning to use the servo rail to power any less tolerant 5 V devices. The other minor quirk is that both I2C ports are only pinned out on the small JST-GH-xP side facing connectors above the main power solder pads. These are admittedly neat connectors which are somewhat commonplace among higher-grade hobbyist components, but add a bit of effort to the assembly and reduce flexibility for cable routing. The position near the location ideal for a chunky main power input capacitor also proved a bit tricky in a very space constrained airframe like this one.

The other improvements from my previous ASW28 are the change to mLRS as a command link, as with all my ArduPilot vehicles, which gives access to transmitter telemetry via the Yaapu script as well as the complete MAVLink data set on a GCS if required, and moving the VTX to the tail to increase RF separation. I should have swapped places between these components however, as the mLRS 900 MHz dipole would have been much better suited to being vertical in the tail and instead has been forced to lie horizontally just behind the battery tray. Best practice aside, the ranges I will fly at mean the dB loss from cross-polarisation won't pose an issue. The VTX is a recycling of my old analog transmitter and Ratel camera mounted in the tail, with the mount designed with the option to upgrade to digital in the future.

Altogether, the airframe is a little heavy in the hand, but for now I am using my trusty (and oversized for soaring) 4S 3300 mAh LiPos. With the battery toward the back of the standard tray, the aircraft balances neatly just behind the factory CG marks, with the option to go further back as I chase the optimal balance. With a total mass of 1.65 kg and an estimated wing area of 0.349 m^2, this places the wing loading at about 47 N/m^2, which isn't terrible.

| Component       | Weight /g      |
| :------------   | ---------:     |
| Fuselage        | 840            |
| Wing (Port)     | 247            |
| Wing (Star)     | 230            |
| Battery (4S 3300 mAh)  | 337     |
| Total           | 1654           |

I sprayed the factory red components with my favourite melon yellow and built the avionics and wiring looms out onto the same tray templates I had made for the previous iteration. All was very straightforward, except the manual does not specify throws, nor had I noted them down from the previous build, but I settled on the throws in the table below. I also hadn't kept my ArduPlane config from the previous iteration so had to tune from scratch again. 

| Surface       | Deflection /mm     |
| :------------ | ---------:         |
| Aileron       | 15                 |
| Elevator      | 10                 |
| Rudder        | 30                 |
| Flaps         | 20                 |

## Flying

The maiden flight was the smoothest I have ever had, with a graceful auto takeoff, climb out, and loiter on the default tune. In FBWA, the model feels very good to fly and quite nimble considering the span, so I have not made any attempt to change the tune - besides, the main reason for the model is to chase thermals in manual. Two minor issues have cropped up, the first of which is a suspect wing connector which sometimes fails to connect the airspeed sensor data lines, which will be replaced in due course. The other is the stock ESC running into overcurrent protection at full throttle and fast ramp up, which I have mitigated with a maximum throttle limit and minor slew rate adjustment in ArduPlane. Perhaps in the future I will swap out the ESC for something more performant and with a few modern features like telemetry.

![Image 1](DSC02768.jpg)
![Image 2](DSC02773.jpg)
![Image 3](DSC02775.jpg)
![Image 4](DSC02777.jpg)

The soaring performance is just as good as I remember from my previous version. Even at this slightly increased weight, it will happily catch a light thermal and follow it as far as needed. On a few occasions I was finding it hard to lose lift for landing, even with full crow brakes! I've only been free on a few good days so far this year, but no doubt there will be plenty of good soaring to come this summer.

## Soaring

I didn't get the chance to investigate ArduPlane's autonomous soaring too much on the last iteration, so I plan to test it out this time. To that end, I performed some initial glide tests to complete the helpful drag characterisation spreadsheet linked on the ArduPlane wiki. I picked a day with a little more breeze than I would have liked, but gathered data at several glide airspeeds from 9 to 18 m/s whilst gliding straight in direct crosswind. At 9 m/s the aircraft had the propensity to oscillate quite noticeably in pitch, so I have constrained the minimum glide speed to 10 m/s going forwards. The resulting glide and drag polars are shown below.

![Glide polar](glide_polar.png)
_Resulting glide polar for the ASW28_

![Drag polar](drag_polar.png)
_Resulting drag polar for the ASW28_

The results of airspeed and sink rate were averaged for each test over the duration of unpowered descent, which was approximately 120 m in all cases. The average sink rate was then plotted against the airspeed in the first figure. The calculated lift and drag coefficient for each test case was then plotted in the second figure. Clearly, the data is not perfect, likely owing to the wind conditions, so the test could be repeated on a still day to improve the results. In particular, the drag bucket seems to be forming in the drag polar but the estimated fit line seems influenced by the possibly erroneous higher lift coefficient results. The test method itself is also not perfect, but an estimate of the glide characteristics can be made regardless, which informs the ArduPlane soaring controller for soaring tests in the future.

| Parameter          | Value         |
| :------------      | ---------:    |
| SOAR_POLAR_K       | 75.9          |
| SOAR_POLAR_CD0     | 0.031         |
| SOAR_POLAR_B       | 0.080         |