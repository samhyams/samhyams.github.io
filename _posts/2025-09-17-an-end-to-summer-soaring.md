---
title: An End To Summer Soaring
date: 2025-09-17 21:35:22 +0100
categories: [Projects, ASW28]
tags: [uas, uav, ardupilot, arduplane]     # TAG names should always be lowercase
description: Reviewing the summer's thermal soaring activity
toc: false
media_subpath: /asw28/
image:
  path: 20250824_131822.jpg
  alt: ASW28 ready for flight
---

I'm sat at my desk right now, after what has been a wet, windy, and generally miserable start to September to kick off the autumn - and with it, the definitive end of soaring season here in southern England. Building my new ASW28 was a fantastic move as I've spent many days over the summer making the most of the thermic weather we've enjoyed as the temperatures rose, and the performance has been better than my previous model ever managed, managing to unintentionally set new endurance and height personal records. Indeed, since maidening the aircraft in April, I've put a total of just under 16 flight hours on the airframe, and although I don't have an exact figure, I imagine a good 95% of that flight time was spent with the motor off. This includes my current record single gliding flight of 2h 22m.

![ASW28](20250824_131822.jpg)

Sadly, I don't have comprehensive logs of any flights as I overlooked the orientation of the flight controller during installation and the micro SD card is inaccessible without disassembly. The log files are too large to reliably download via ArduPilot's MAVFTP. Once I realised this, I set up EdgeTX's native logging sensor functionality to at least get basic flight log data for inspection. As I'm using mLRS for my C2 link, I have access to the full suite of standard ArduPlane telemetry packets on the transmitter. Not only can I view these in real time using the Yaapu telemetry scripts, but as mLRS passes them to EdgeTX as CRSF sensors they are available to the EdgeTX logging function, which I use to log all sensor values at 2 Hz when the vehicle is armed. These logs are crude relative to the full ArduPilot .bin logs, but good enough to assess glide performance and review basic flight data.

I didn’t really intend to try ArduPlane's auto soaring feature this time as I built the model for manual thermal chasing, but one day my FPV feed was playing up so I decided to give Ardu Soar a try. In my previous ASW28 blog post I discussed how I determined some of the glide polar values experimentally and updated the aircraft config with these values. Initially, the auto soar performance was so-so. The aircraft would detect some lift and then proceed to meander, without managing to get much from the thermals - granted, the wind was a steady 5-10 m/s throughout the flight but there were some strong thermals in the area worth chasing. I tweaked a few of the parameters which helped improve the performance a bit:

- Reduced SOAR_DIST_AHEAD to 1 m which reduced the chance of flying right through the thermals
- Increased SOAR_VSPEED to 0.9 m/s to avoid some more phantom lift
- Increased SOAR_MIN_THML_S and reduced SOAR_MIN_CRSE_S to try make it try thermalling a with more persistence

This flight I was flying around in CRUISE and the performance looked like this (where green areas are in THML - aka autonomous soaring - flight mode):

![Auto soaring 1](soar_plot_1.png)

For reference, at or below SOAR_ALT_MIN (50 m AGL) the motor will activate and the aircraft will climb. Then, once it reaches SOAR_ALT_CUTOFF (250 m AGL), the motor is stopped and does not activate again unless the aircraft falls to SOAR_ALT_MIN. Between flights I updated a few more parameters to make the thermal loiter more aggressive:

- Increased SOAR_THML_BANK to 40 degrees
- Reduced WP_LOITER_RAD to 40 m

Despite it getting later in the day (take-off at almost 4 pm), the thermal performance was much better now, and I set the aircraft off in AUTO following a 500 x 500 m square around me. I eventually had to end the flight because I needed to go home, but the best period was a 53 minute glide (starting at ~1400 seconds below) which far exceeded my expectations and I think there’s more performance to get out of it. The benefit of THML is that it seems much better at staying within the thermal vs. getting blown downwind which I often suffer when thermalling in manual nav modes on windy days.

![Auto soaring 2](soar_plot_2.png)

I found the performance surprisingly good, and came away with a couple of minor thoughts for improvements with the default Soar behaviour:

- A MAVLink message is provided when entering THML, but it would be nice to also get a message when the motor is triggered at both SOAR_ALT_MIN and SOAR_ALT_CUTOFF
- Perhaps a delay between reaching SOAR_ALT_CUTOFF and allowing THML to be entered - I think the aircraft decided the residual climb rate from powered climb was thermal lift in a few instances (need full .bin logs to assess this properly)
- Sometimes the aircraft seems to leave THML mode despite still being in lift (need full .bin logs to assess this properly)
- Separate the THML loiter radius from WP_LOITER_RAD

I intended to take a look at Mavproxy’s soar module for viewing thermal position estimation, and try the SoarNav Lua script which adds much more complex soaring behaviour. Sadly, other commitments on the final opportunities for soaring days and the start of autumn weather put paid to these plans, so they are on the back burner until the thermals return in 2026.