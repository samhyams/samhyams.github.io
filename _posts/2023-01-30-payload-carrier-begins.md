---
title: Payload Carrier Begins
date: 2023-01-30 20:13:00 +0000
categories: [Project Updates, Orca]
tags: [uas, uav, autonomous, ardupilot, arduplane]     # TAG names should always be lowercase
description: Designing the systems of my new payload carrying autonomous RC aircraft
toc: false
---

My ASW28 is now no more after an unfortunate midair collision during a sloping session last October. The other aircraft collided head-on and tore off the entire tailplane of my glider, leading to a swift game of pin the tail on the donkey with the Westbury white horse!

![End of the ASW28](/asw28/rip_asw28.png)
_The final moments of my ASW28 recorded in the datalog_

All the internal components survived, so it's time to repurpose them into my next platform. I need an easy to launch airframe with plenty of internal volume for batteries and payloads so that I can change the use case between long range endurance and large payload carrying capability, and after a short search I've settled on the AtomRC Killer Whale. Although it leaves something to be desired as far as aesthetics go, it's a reliable foamie that has lots of internal space with a twin puller motor configuration and a high wing which will allow for ground take-offs.

The systems mostly use components lifted straight from the ASW28, including the Matek F405-WSE flight controller, 5.8 GHz VTx, 433 MHz telemetry module, and RC receivers. The powertrain setup is new and uses the recommended 2306 1400kV motors and 7x4" triblade props, with Holybro Tekko32 45A ESCs running BlHeli32 firmware. This should allow for ESC telemetry to be returned to the flight controller. Separation of RF devices has been included to prevent issues arising and requiring retrofits, this results in the VTx placed on the port wingtip, RC Rx on the starboard wingtip, and the C2 telemetry near the tail. Any additional RF devices used in the future can occupy the main fuselage bay or the nose.

![Orca System Diagram](/orca/orca_system_diagram.png)
_Initial system diagram of the new Killer Whale build_