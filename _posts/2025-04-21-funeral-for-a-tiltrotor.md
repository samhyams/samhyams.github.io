---
title: Funeral For A Tiltrotor
date: 2025-04-21 15:52:31 +0100
categories: [Projects, Mini Caelus]
tags: [uas, uav, vtol, transition, autonomous, ardupilot, arduplane]     # TAG names should always be lowercase
description: Analysing the final flight of my ducted fan tiltrotor
toc: false
media_subpath: /mini_caelus/
image:
  path: final_flight_wide1.jpg
  alt: Final transition still image
---

It's been two years since I finally flew the maiden transition of my custom tiltrotor, _Mini Caelus_, but now the story comes to an end. Since that post, I haven't discussed it too much here as I only flew it once every few months for fun. The troublesome powertrain, high wing loading, extensive setup time, and penchant for calm days made flying sporadic, despite the huge support it always gained from spectators at my flying club. I had refined the tune over time to produce a very reliable and smooth transition in both directions, with the only bugbear remaining the poor yaw performance in hover. This last issue was caused by actuator lag from the vectored yaw tilt servos, and due to the lack of battery capacity for hover tests (one very small and run down 6S LiPo) and the fact that the oscillations were stable, I simply overcame the yaw problems by not inputting yaw commands to prevent any PIO. The battery issues had become so severe by the end that in the winter months it was necessary to run the motors at half power on the ground for 30 seconds before takeoff, otherwise the battery sag from the cold would make the aircraft unable to hover!

My attitude to the airframe was that once I had the first major crash, I would retire it, as a result of many key airframe components being parts I can't easily make a new copy of. The wings are hot wire cut XPS foam and I don't have access to a CNC hot wire cutter any more, the fuselage ribs are CNC machined XPS foam and I don't have access to a 3-axis CNC machine any more, and the fuselage skins with selective foam sandwiches require a vacuum pump, which I also don't have access to any more.

## Final Flight

The final flight took place on a calm early evening with clear skies and temperatures dipping just below 10 degrees. The aircraft was struggling to hover, and after one failed attempt to get to transition height I warmed up the batteries for a short while before attempting transition again. With the hover performance being lethargic due to the marginal power from the front duct, it had become standard procedure to begin the transition at a mere 10-15 m AGL, so once I eyeballed the height to be about right, I started the transition. Only a few seconds after starting, a strong uncommanded right yaw developed. As this was during transition, several factors made this a significant problem:

- Yaw output is performed by using differential thrust, so the port motor spools up and the starboard motor spools down
- The motors are still partially tilted, so motor thrust still has a significant upward component
- The airspeed is too low for control surfaces to meaningfully affect attitude

This resulted in the aircraft yawing through 90 degrees and rolling in excess of 110 degrees to starboard in less than 1 second, as well as losing 30% of height above ground, at which point all control was lost. The low transition start and marginal hover thrust meant height meant recovery in QSTABILIZE was impossible, and the vehicle snapped left before impacting the ground on the left wingtip and canard.

{% include embed/youtube.html id='S8TH_5hfKe8' %}

Looking into the logs, the cause appears to be a yaw command driven by a switch to EKF3. The log from the crash is shown below, where I have plotted the desired and actual yaw from the attitude controller (yellow and green) alongside some other data for context; height AGL (red), airspeed (purple), and RC yaw input (blue). The background colour denotes the flight mode, the initial red section is QSTABILIZE and the orange is FBWA. The transition begins as soon as the mode changes to FBWA. Additionally, the logged messages are shown in the bottom right corner with timestamps.

![Crash log](mc_log_end1.png)
_Log from the crash_

The initial hover in QSTABILIZE shows the tendency to oscillate in yaw, looking slightly unstable in this instance, but this is standard for the airframe. The initial part of the transition begins as expected, with a gradual increase in airspeed and yaw oscillations dying off from natural weathervaning. At 09:52, EKF3 resets itself to the new GPS course, and this immediately causes a large change in commanded yaw angle, which the aircraft unfortunately does a good job of meeting. My standard procedure in transition is to not touch touch yaw or roll on the sticks, and the log shows the yaw input was not touched until the very end. The rest is described above and ended the flight. This still leaves a few questions: how has this not happened in a a previous flight, and why does the EKF command affect the yaw output in a transition between QSTABILIZE and FBWA?

## Finding the cause

Plots from previous logs are shown below for context, from several random successful transitions in past flights. 

![Example transition 1](mc_log_005.png)
![Example transition 2](mc_log_009.png)
_Example successful transitions_

One factor which was immediately clear was that no previous flight had an EKF3 reset during transition, and indeed deeper examination showed that EKF3 was not enabled in the ArduPlane configuration for earlier flights. The crash flight was the first flight after upgrading firmware from 4.3.5 to 4.6.0-beta4, as I had upgraded my control link to mLRS and wanted to take advantage of the RC_CHANNELS feature in the 4.6.0 dev branch. I hadn't realised that at some point after 4.3.5, EKF3 had been made enabled by default in stock firmware downloads, and my previous 4.3.5 config had had it disabled. As I neglected to do a param compare after upgrading, satisfying myself with more in-depth pre-flight check and range test, this change in EKF reliance slipped through into flight.

All previous successful transition logs show small adjustments to the desired yaw during transition and the aircraft yaw direction following it (noting the differences in axes scaling). Clearly on transition completion, once the minimum airspeed is met and FBWA control logic is solely used, the desired yaw drops to zero and is no longer used by the control system. Conversely, the first image above shows that desired yaw is used in QSTABILIZE during a period of RC yaw input, so during the transition the mixing of QSTABILIZE and FBWA logic still applies desired yaw, as seen in the crash.

## Lessons learned

The main factor in this crash was the unintentional enabling of EKF3 leading to a significant change in vehicle yaw angle estimation and corresponding yaw output during transition. This vehicle did not have a dedicated compass, so once the GPS course was determined during transition the yaw estimate changed by 130 degrees. If a compass was present, the initial yaw estimate should have been accurate enough to avoid this yaw estimate reset event, and the vehicle likely would have continued as normal. Most of my other vehicles have a compass-enabled GPS and I only install compass-enabled GPS sensors in new builds, so this shouldn't be an issue going forwards.

There are also several human factors considerations which contributed to the failure. Complacency on updating firmware and not performing a full param compare meant the enabling of EKF3 was missed. The low transition height of 10 m, although practiced and proven acceptable on many previous flights, does not allow for sufficient time to correct serious transition problems. Replacing old components such as the 6S duct battery to ensure the vehicle can perform at its best should always be done where possible, or the aircraft should be grounded.

## Goodbye, old friend

The damage was actually less severe than I expected from the impact. The port wing tip foam was crushed and mangled, the port canard was destroyed and the spar broken in two, and the nose partly caved in with a large crack running up the front of the fuselage. The front landing gear was also destroyed and the servo broken. Altogether, whilst it would be possible to fix these issues with enough effort, I think this serves as a fitting end to the project. Thanks for the memories, and see you in the next life :(

![Rest in peace](goodbye.jpg)