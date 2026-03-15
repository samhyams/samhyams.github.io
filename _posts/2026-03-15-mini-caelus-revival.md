---
title: The Short-Lived Mini Caelus Revival
date: 2026-03-15 10:54:10 +0100
categories: [Projects, Mini Caelus]
tags: [uas, uav, autonomous, ardupilot, arduplane]     # TAG names should always be lowercase
description: Building a launcher just to discover new ways to crash
toc: false
media_subpath: /mini_caelus/
image:
  path: DSC02825_edited.jpg
  alt: Ready for launch
---

After I crashed my tiltrotor last year, I had written off the airframe and though of the project as over for good. But as time passed, I thought it would be relatively straightforward to repair the wings and dust off my original testbed fixed-wing fuselage. After seeing some successful bungee ramp launches at the flying site a few weeks ago, I decided to combine the aircraft refurb with building a cheap ramp and launch system which I can reuse for other projects in the future. Plus I've been quite busy with moving and starting a new job and this seemed like an easy project to get back into it!

The first thing to look at as the wings themselves. As I thought, the damage was actually quite minor, with only the port wing actually needing any repair at all. I stripped off the old lite solarfilm and grabbed the 5 minute epoxy to effect repairs, reinforcing the large midplane crack with some bamboo skewer segments as rebar embedded in the epoxy. I replaced the winglets with the originals and checked over the control surfaces, with all appearing to be in order. The fuselage needed no work at all, and I printed and assembled a quick fixed motor mount design to replace the tilting mounts. I used the original tiltrotor electronics throughout, with the flight controller, airspeed sensor, ESCs, mLRS receiver, motors, and old folding 8x4 props making an appearance. Some of these components are on their last legs after a few crashes though, as the flight controller was giving a lot of minor EKF warnings and the airspeed sensor appeared to be completely non-functional. Nothing to prevent flying as a basic RC plane, but not parts that I will use in the long term or on other projects. I was tempted to tidy up the model with a fresh livery, but decided to check it still flew before committing the time and resource - a decision which was about to pay off...

![Wing repair 1](20260308_155937.jpg)
_Removing the old covering film - with some difficulty_

![Wing repair 2](20260308_162054.jpeg)
_Second time around repairing wingtip damage_

![Wing repair 3](fix-wing.jpg)
_Repairing the major crack in the port wing_

The launch ramp was a quick and dirty design which turned out rather nicely. I raided B&Q for some 8mm bungee cord, tied a loop in each end, and attached a few metres of kevlar string leftover from my _other_ bungee to one end. In the kevlar line I made a loop at the end to attach to the pedal release and another loop a little bit down from the end to interface with the vehicle. The ramp itself was some cheap plastic pipe cut to length and connected with some 3D printed pipe joiners to give a 10 degree ramp angle and about a 1500 mm ramp length. Each foot has an adjustable threaded inner to allow for levelling on rough ground, and tent pegs are used to secure the ramp and pedal to the ground. The pedal was another simple 3D print, with a locking pin included on the bungee release pin to prevent accidental launches.

![Launch ramp pedal](bungee-pedal.png)
_Bungee pedal CAD_

Yesterday was a surprisingly nice day - arguably the best of the year so far - so I ventured down to the Deverills and began to set up. There was an unexpected appearance of this year's Team Bath Drones undergrads at the flying field and it was great to chat to them as we went about our projects throughout the day. A few even recognised my aircraft, as the original drawing from 2018 is apparently still on the wall in the lab in Bath! One of their aircraft was on 900 MHz mLRS too, and we seemed to have issues operating together, but we didn't dive into the _why_ and instead solved it with some old-fashioned temporal diversity. The winds were light and changing direction often, so I set up the bungee into the prevailing as dictated by the flight line and got to work setting up the aircraft. With a break in the wind I pressed the pedal and away we went.

![Pre-launch 1](DSC02822_edited.jpg)
![Pre-launch 2](DSC02825_edited.jpg)
_Ready for launch_

The launch was a little unexpected. The initial pull was very weak and I didn't think it was going to go, but once off the ramp the pull increased and the aircraft was in the air. My surprise meant there was a fair amount of overcompensation in the initial launch, but it settled down and I flicked it into FBWA just after the initial climb out and turn. The very start of the launch off the ramp also had a significant right roll, likely caused by a combination of ramp/bungee misalignment and slight left crosswind so these are best avoided in future launches.

The performance was OK in FBWA but not spectacular. I didn't try tuning it due to the distraction of the many inquisitive spectators, instead flying a few minutes of lazy circuits and low passes. Despite the tune, the aircraft still looks great in the air! The sound was not so great however, with a noticeable grinding noise coming from somewhere on the motor side and getting worse as the flight went on. I decided to continue for a few more minutes before landing, which was a poor decision. Just before beginning a downwind turn to return to the flight line, the port wing appeared to fold and detach from the fuselage, followed by the various parts plummeting into the short crop below - to a chorus of amused gasps from myself and my fellow onlookers! I was unhappy thinking that it had been a structural failure of the wing, something I've not had happen on any of my custom models to date, but when I found the wreckage it told a different story.

The port wing had cleanly separated from the fuselage and the airframe was surprisingly mostly intact, aside from a few minor broken bits of plastic. The leading theory is that the grinding noise and accompanying vibration worked the port wing retention bolt loose which then allowed to wing to depart from the aircraft. As for the source of the grinding noise, it's a little uncertain. Running up the motors post-crash, the starboard spins quietly but the port motor exhibits the now characteristic noise and vibration. The exact reason behind this is unclear, but it's most likely either damage to the motor from the original tiltrotor crash which went unnoticed, or misbalancing of the folding props which do seem a bit tight on the centre piece. Regardless, the root cause is not too important as I definitely won't be flying these motors or props again. As for the airframe however... it has somehow survived yet another catastrophic crash and so I may well patch it up to fly another day. Time will tell!

{% include embed/youtube.html id='7gktGNL-slg' %}
