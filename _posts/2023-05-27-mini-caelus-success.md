---
title: Mini Caelus Success
date: 2023-05-27 18:52:00 +0100
categories: [Project Updates, Mini Caelus]
tags: [uas, uav, vtol, transition, autonomous, ardupilot, arduplane]     # TAG names should always be lowercase
description: Maiden flight of my custom built tiltrotor VTOL flying wing
toc: false
media_subpath: /mini_caelus/
image:
  path: DSC02089.jpg
  alt: Mini Caelus rear shot
---

The tiltrotor airframe has always suffered from a lack of front duct power on 4S, causing pitch control to be difficult and unreliable in hover. The first upgrade to the airframe in April was a change of motor and prop configuration in the duct to move to 6S, with coaxial T-Motor F80s on 5x3 inch triblades providing a theoretical maximum 3.5+ kg of thrust at over 80 amps. The duct is powered from a separate 6S LiPo with the rear motors, power electronics, and avionics powered from the remaining 4S LiPo.

VTOL flight issues were immediately solved with this change, so there were no more roadblocks to the first transition flight.

{% include embed/youtube.html id='2u5dwQc_tX0' %}
_First transition_

## Making Iterations

The first transition was not perfect but forward flight was achieved! After a few circuits in fly-by-wire, I lined up for landing, which worked without issue - although I overshot the intended landing point by quite a distance due to excess speed.

I hadn't expected the aircraft to survive the first transition, so now the question was - what next? The first problem to tackle was the excessive rolling seen during the forward transition. Analysis of the logs showed this was due to roll-yaw coupling during transition before the motors have fully transitioned to forward flight. This was solved by reducing Q_TILT_MAX, the angle that the tilt motors are set to during transition, from 45 to 35 degrees, which cleared up the majority of the attitude oscillations for further transitions. The remainder can be attributed to minimising the yaw rate at the initialisation of the transition manoeuvre.

Other minor transition parameter adjustments included:

- Q_TILT_RATE_UP: the tilt motor movement rate during backward transition, increased from 40 to 90 deg/s
- Q_TILT_RATE_DWN: tilt rate for forward transition, kept at 40 deg/s
- Q_TRANSITION_MS: the minimum transition time before mode is FBWA, reduced from 1500 to 1000 ms

VTOL yaw response was slightly tuned to accommodate Q_TILT_YAW_ANGLE (the maximum angle from vertical for vectored yaw) being increased from 20 to 30 degrees, since there is excess power on the rear motors and this is a large driver of VTOL yaw responsiveness.

Subsequent flights aimed to characterise the CG location, flight envelope, and tune the FBWA control loops. In manual, the canard provides excessive control authority at the current +/-15 degree maximum deflection, and can cause an immediate stall into flatspin if not carefully used! One of these occurrences caused a 6G manoeuvre load, but in every case the aircraft was recovered with a transition to VTOL mode. Future work will look to reduce the canard deflections and to remove the elevon mixing from the wing control surfaces, transforming them into pure ailerons and improving efficiency of the main wing.

![Mini Caelus 1](DSC02094.jpg)
![Mini Caelus 2](20230416_174741.jpg)
![Mini Caelus 3](DSC02096.jpg)

{% include embed/youtube.html id='SIM26qsM4d0' %}
_The result of not being cautious using the canards in manual!_

## Crashing

After more than 10 flights, the aircraft was flying reliably and was handling well in fly-by-wire, with an initial attempt at FBWB mode showing a mediocre efficiency of 110 mAh/km at 18 m/s airspeed. This will be the focus of future flight tests, but things never go according to plan, as was shown in the next flight.

The first attempted transition on the next day ended abruptly with a crash. Only moments after beginning transition, the duct shut off and the aircraft somersaulted to the ground. My first thought was to blame a defective duct current sensor, which I had just installed for this flight, but closer analysis of the log showed the true cause.

![Crash log](crash_log.png)
_Data log from crash flight_

The duct (C9, green line) was in fact commanded to shut off, but why? This flight had intended to test efficiency in FBWB, so I had reduced the FBW minimum speed to 10 m/s (from 12 m/s) to allow slower speeds to be flown. However, due to the wind speed on the day, and the fact that the aircraft was moving forward in hover before transition was initiated, the measured airspeed hit 10 m/s at almost the exact moment that transition was initiated. This caused the aircraft to use the minimum transition time of 1 second, at which point the duct throttle was reduced to zero.

However, the tilt motors have a maximum tilt rate of 40 deg/s and therefore were not fully rotated to forward flight when transition was completed (see C4), causing an uncontrollable nose down pitching moment. My assumption in the moment that the duct had failed, rather than transition completing, meant I didn't revert to VTOL flight mode before the impact.

![Crash aftermath](20230527_173709.jpg)
_Aftermath of the transition crash_

Luckily, the damage was reasonably minor, with the wings and fuselage in almost perfect shape. Fixes to the winglets, front duct motor cradle, front leg holder, and booms are all that are required to get the aircraft flightworthy again. Only the booms present a challenge as they will need to be redesigned to be manufacturable with the equipment I have on hand now.

Parameter changes to prevent this from happening again include:

- Increasing Q_TRANSITION_MS to 2500 ms, to allow the tilt motors to fully rotate before the duct is throttled down
- Increasing the FBW min speed back to 12 m/s
- Changing operational procedures to not fly in winds above 5 mph and not move forward during hover

![Mini Caelus](DSC02085.jpg)