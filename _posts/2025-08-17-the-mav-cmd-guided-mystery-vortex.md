---
title: Decoding the MAV_CMD_GUIDED mystery vortex
date: 2025-08-17 16:06:12 +0100
categories: [Other]
tags: [uas, uav, ardupilot, arduplane]     # TAG names should always be lowercase
description: Testing the hidden command overrides for speed, altitude, and heading
toc: false
media_subpath: /beacon-rth/
image:
  path: heading_comparison.png
  alt: Comparison of commanded course over ground heading and magnetic heading
---

I've had a busy few weeks with the recent great weather in the south of England, having excellent luck with soaring my ASW28 and even trying out ArduPilot's automated soaring functionality with decent success. But with a few minor complications cropping up, it seems like a full post on that will have to wait until soaring season is over.

In the meantime, I recently had reason to look back at my Beacon RTH project and think about some of the decisions made about the functionality in that project. The navigation control was done by extrapolating a waypoint in the distance along the desired heading, and setting the aircraft to fly to that point. This was done because, at the time, I didn't think ArduPlane natively supported a direct heading command input to override navigation. Digging a bit deeper however, a few useful MAVLink commands do exist for control of speed, altitude, and heading in GUIDED mode. These commands are not part of `common.xml` but are instead under `ardupilotmega.xml`, meaning they should apply to any APM-based vehicle, including ArduPlane. These commands are as follows:

- MAV_CMD_GUIDED_CHANGE_SPEED ([43000](https://mavlink.io/en/messages/ardupilotmega.html#MAV_CMD_GUIDED_CHANGE_SPEED))
- MAV_CMD_GUIDED_CHANGE_ALTITUDE ([43001](https://mavlink.io/en/messages/ardupilotmega.html#MAV_CMD_GUIDED_CHANGE_ALTITUDE))
- MAV_CMD_GUIDED_CHANGE_HEADING ([43002](https://mavlink.io/en/messages/ardupilotmega.html#MAV_CMD_GUIDED_CHANGE_HEADING))

So in theory we can fly around in GUIDED mode and have control of our airspeed, height, and heading all at the same time. Let's give it a try in ArduPilot's SITL environment.

## Speed

In pymavlink, the MAV_CMD_GUIDED_CHANGE_SPEED command can be send using a command long message, following the MAVLink definition:

```
set_speed_message = dialect.MAVLink_command_long_message(
        target_system=plane.target_system,
        target_component=plane.target_component,
        command=dialect.MAV_CMD_GUIDED_CHANGE_SPEED,
        confirmation=0,
        param1=0,       # Set airspeed (0) or groundspeed (1)
        param2=speed,   # Desired speed
        param3=ar,      # Acceleration
        param4=0,
        param5=0,
        param6=0,
        param7=0
    )
```

Param 1 uses the SPEED_TYPE enum to set whether this is airspeed or groundspeed. In Plane, the command fails if groundspeed is requested, which is not entirely unsurprising as it would be very unusual to request a fixed wing aircraft to fly a set groundspeed. Requesting airspeed works as intended.

## Altitude

Similarly for altitude changes, following the MAVLink definition the desired altitude (AMSL) and climb rate can be set within the request:

```
set_altitude_message = dialect.MAVLink_command_long_message(
        target_system=plane.target_system,
        target_component=plane.target_component,
        command=dialect.MAV_CMD_GUIDED_CHANGE_ALTITUDE,
        confirmation=0,
        param1=0,
        param2=0,
        param3=cr,      # Climb rate
        param4=0,
        param5=0,
        param6=0,
        param7=alt      # Desired altitude
    )
```

The climb rate units are quoted as m/s in the MAVLink documentation, but I don't think the implementation behind the hood is quite right as it doesn't seem to match this rate in SITL.

## Heading

The thing I'm most interested in is heading commands, as this is useful for GPS-denied heading based navigation. Again, the message is constructed following the MAVLink definition:

```
set_heading_message = dialect.MAVLink_command_long_message(
        target_system=plane.target_system,
        target_component=plane.target_component,
        command=dialect.MAV_CMD_GUIDED_CHANGE_HEADING,
        confirmation=0,
        param1=ref,     # Set course over ground (0) or raw heading (1)
        param2=hdg,     # Desired heading
        param3=hr,      # Max heading rate
        param4=0,
        param5=0,
        param6=0,
        param7=0
    )
```

The heading rate must be populated, as setting to zero will result in no heading change. Basic testing shows that the heading reference, i.e. whether the aircraft uses the input heading as a course over ground (ref=0) or nose magnetic heading (ref=1), works as intended, with the latter being useful for a basic GPS-denied navigation solution. More advanced solutions may provide a continuous course over ground estimation, in which case that heading command reference can continue to be used.

![Heading comparison](heading_comparison.png)
_Comparison of commanded (a) course over ground heading, and (b) magnetic heading_

## Summary

So overall, nothing groundbreaking, but perhaps useful to somebody looking to implement speed, altitude, or heading control in a MAVLink system, and certainly an important functionality to make use of the next time I use offboard control in ArduPilot.