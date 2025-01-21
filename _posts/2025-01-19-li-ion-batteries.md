---
title: Lithium Ion Batteries
date: 2025-01-19 16:16:19 +0000
categories: [Other, Orca]
tags: [uas, uav, ardupilot, arduplane, battery, liion]     # TAG names should always be lowercase
description: Building and testing a 4S2P Lithium Ion battery pack
toc: false
media_subpath: /battery/
image:
  path: DSC02655.jpg
  alt: Final stages of battery construction
---

Years ago I had made a little 2S Lithium Ion (Li-Ion) battery pack from two 18650 cells to power my ZOHD Dart. The stock setup of a 1000 mAh LiPo managed a 20 minute endurance, but using the 3500 mAh Li-Ion pack stretched the endurance to about 80 minutes - impressive for a sub-250g aircraft! The benefits of a more power dense battery chemistry and drawback of a lower peak current draw compared to LiPos are well known to anyone in the UAS space, and I wanted to see how far I could push the endurance of my *Orca* using a big Li-Ion pack.

A fellow flying club member offered a set of cells to use for a new pack, some Molicel P42A 21700s, which boast a 45 A continuous discharge capability and 4200 mAh capacity. With 8 cells, a 4S2P battery pack with a combined capacity of 8400 mAh and discharge rating of 90 A can be constructed.

## Building the battery pack

The requisite components to build a pack are quite simple. Besides the obvious wire and connectors, some nickel strips, kapton tape, adhesive barley paper, and heatshrink are all that are required. Without a spot welder, I chose to solder the cells together. The first step was to print a jig to hold the cells together in shape, lightly sand the terminals, and tin the terminals with some solder. It's important to use a high iron tip temperature as the battery provides a large thermal sink to the terminals, but the temperature of the cell can't rise too much or you risk damaging or destroying the cell.

![Tinning battery terminals](DSC02651.jpg)
_Tinning the battery terminals with solder_

Next is to attach the nickel strips. I bought some 0.15 mm thick plates designed for this application, and they feature gaps near the centre of the cell to aid with reflowing the solder under the strip. I attached some balance leads with standard colour coordination to each cell terminal point.

![Adding nickel strips](DSC02652.jpg)
_Adding nickel strips and cell balance leads_

I added a small amount of hot glue between adjacent cells to maintain the pack structure, then added the main power wires to the end terminals. I would have prefered 14 AWG for this pack, but used the slightly heavier 12 AWG that I had on hand.

![Adding main power leads](DSC02653.jpg)
_Adding main power leads_

After adding plenty of kapton tape to the exposed terminals as necessary, I covered the long side and underside of the cells with a layer of adhesive barley paper, which adds a small amount of non-conductive protection to the pack. I then cut the wires to length and attached the XT60 main power connector as well as crimping the JST balance plug.

![Adding barley paper](DSC02654.jpg)
_Barley paper added to the side and bottom of the pack_

![Connectors added](DSC02655.jpg)
_Power connectors added_

Finally, I heatshrunk the entire pack and compared the results to a normal 3300 mAh 4S LiPo.

![Final pack](DSC02656.jpg)
_Final pack_

The Li-Ion pack weighs 577 g in total. Considering each of the eight cells weigh about 65 g, the overhead for the rest of the material is 57 g, or almost exactly 10% of the total weight. With a nominal maximum capacity of 8400 mAh, the pack weight efficiency is about 0.069 g/mAh, which compares to the 0.102 g/mAh of the LiPo above.

## Flight Endurance

Before flying, I stress tested the pack by running on the ground at full throttle for a few minutes. This only draw a maximum current of about 40 A, but this is the only aircraft I plan to use the pack with. The battery fits very snugly in the battery bay and balances the same as my normal setup of two 4S LiPos.

I flew a single endurance flight. It was a cold day (temperatures somewhere between 0 and +5 degreed C) and I climbed to 120 m AGL and flew a wide circular loiter with 250 m radius at 15 m/s for the duration. This flight profile gives an almost wings-level attitude for the flight, with an average roll angle below 2 degrees. At takeoff, the cell voltage was 4.17 V, and I landed once the cell voltage reached 3.2 V, which gave a total flight endurance of 95 minutes and flight distance of 82 km. The current monitor is a bit out of calibration, claiming a use of 8500 mAh from the 8400 mAh battery, but assuming we used the full 8400 mAh this puts the aircraft efficiency at 102 mAh/km.

![Battery log](battery_log.png)
_Logged results of battery voltage and current draw during the endurance flight_

This is a fantastic result but there is room for improvement. Ensuring a full 4.2 V per cell on launch and better exploiting the high depth of discharge the Li-Ion cells provide could provide a noticeable benefit. Indeed, the P42A datasheet shows the cell voltage dropoff is not significant until about 3.0 V per cell at the cruise current draw of 5 A or less. Similarly, warmer air temperatures can extend the life of the cells by about 5% at best, but this may not be significant in this application due to the self-warming nature of the cells under load.

Moving to a more efficient airframe would significantly boost the maximum endurance. The 102 mAh/km figure compares unfavorably with the ~65 mAh/km previously achieved with my ZOHD Dart. This is the unsurprising result when comparing a single-engined plank wing to a twin conventional aircraft, so perhaps the next step is to build a 4S wing for future endurance flying.