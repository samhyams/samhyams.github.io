---
title: Beacon RTH (Part 1) - minimal SWaP-C GPS denied return to home navigation
date: 2024-10-17 20:05:06 +0100
categories: [Projects, Pinegrove]
tags: [uas, uav, ardupilot, arduplane, pymavlink, gps, gnss, denied]     # TAG names should always be lowercase
description: Beacon RTH for UAS in GNSS-denied environments using LoRa radio ranging
toc: false
media_subpath: /beacon-rth/
image:
  path:
  alt: 
---

Beacon RTH for UAS in GNSS-denied environments using LoRa radio ranging

# Introduction

UAVs commonly use return-to-home (RTH) as a failsafe for situations where the aircraft should return home autonomously, such as a loss of comms or low battery scenario. RTH typically relies on GPS position navigation, with dead-reckoning using compass heading as a backup when GNSS position is unavailable. This dead-reckoning mode is not always reliable, as it relies on accurate wind estimation to account for drift during the return flight, which can cause significant position errors as distance to home increases.

Emerging novel methods of improving GNSS-denied navigation include optical flow and computer vision with a stored topology database to triangulate position. These methods are complex and can require significant weight and power overheads on the aircraft, which impacts endurance and cost. These methods can also be more difficult to perform on fixed-wing aircraft which must always be moving across the terrain.

LoRa radios offer a ranging feature, where the point-to-point distance between two radio modems can be determined using time-of-flight. These radios are very low SWaP-C and therefore have good potential for use as a beacon system. The RF properties are also relatively benign, as they can be used in ISM bands and at low EIRP to prevent EMC issues with other systems on the aircraft. In addition, they are claimed to be able to function below the noise floor which improves resilience against jamming and benefits ultimate range.

By knowing the distance from home, the relative closing speed of the aircraft to the home location can be calculated and compared to previous results to determine the trajectory of the aircraft relative to the home location over time. This can ultimately be used to crudely guide the aircraft back towards the home position. This project doesn't aim to match the accuracy of GPS-guided RTH or even the other GPS-denied solutions mentioned above, but the aim is to autonomously bring the UAV back in the direction of home to a point close enough that the pilot can gain visual, retake control, and land - all without the need for any GNSS-derived input data.

# Investigating LoRa ranging

Inspiration for this project came from reading through the Semtech LoRa documentation whilst working on another project. The documentation describes the in-built radio wave time-of-flight functionality, which can determine the distance between to LoRa radio modems.

An open source arduino library for LoRa radio modules, shared by StuartsProjects on GitHub[1], not only offers an easy way to get communicating between LoRa modems, but also a ready made ranging example (and some testing results!) which can be adapted for custom use. The modifications needed were very minor; I set one ground node as a ‘transmitter’ to broadcast ranging requests and one airborne node as a ‘receiver’ to respond to the ranging request. The ground node then output the resulting distance measurement via serial to a connected laptop where it could be recorded.

![Hardware implemetation of the LoRa air node](air_node_front.jpg)
_Perfboard setup of the LoRa air node_

The radio modems consist of Ebyte E28 SX1280 modems mounted on breakout PCBs to allow breadboarding. The air node fits on a 6x4 cm perfboard with an Arduino Pro Micro and a cheap BEC to provide 3.3 V to the circuitry, keeping the air node self-contained when attached to a power source. The ground node is less permanent, with the same components mounted on a breadboard to be powered from an external power source or USB port.

![Air node testing](field_setup.jpg)
_Air node attached to a tripod for initial testing_

I setup the air node to broadcast from my Orca aircraft and placed the ground node next to my GCS. After flying patterns and orbits for about 20 minutes, I analysed the data to compare the LoRa ranging measurements against the calculated GPS distance from the aircraft, and you can see the results below.

![LoRa ranging test results](distance_comparison_sub.png)
_Comparison of GPS and LoRa ranging distance measurements during flight_

The results were very promising with only a small delta between the LoRa and GPS results, which can easily be accounted for by several factors aincluding GPS accuracy and the aircraft home position vs GCS position offset of a few metres. For the Beacon RTH application, the accuracy shown is more than good enough as the ranging results can clearly be relied on to determine if the UAV is moving away from or towards the home point.

# System Setup

My initial ranging tests took place over a year ago, and I didn’t start my work on the larger RTH solution until summer had turned to autumn this year. Thankfully, I’d documented my previous work well enough to give myself a head start. First off is determining how to setup the system for navigation control in a way which is both performant and safe. 

As mentioned before, the LoRa radio modems are set up for ranging using arduino microcontrollers to provide a serial interface. I switched the roles of the air and ground node, so the air node requests a ranging measurement and outputs the response from the ground node on its serial port. This provides automatic ranging updates at approximately 2 Hz for use by other systems on the UAV. The airborne LoRa node onboard the UAV is connected to a companion computer. The companion computer is also connected by UART to a serial port on the flight controller to provide bidirectional communications, allowing the companion computer to both receive flight data and send control commands. The ground LoRa node is a standalone modem and microcontroller which responds to the ranging requests made by the airborne node.

![Basic System Diagram](system_diagram_basic.png)
_Basic system overview_

# Hardware Implementation

## System Safety

Writing a bunch of requirements leans a little too close to my day job, but given this project involves handing over navigation control to some of my own hastily-written hacked together code, I think it’s best practice to stipulate a few key safety requirements. Here are three very high level requirements which must be met by the system as a whole:

- RC commands shall always override the novel payload
- UAV failsafes shall always override the novel payload
- The novel payload shall not command the UAV to make an unsafe manoeuvre

To meet these demands, several key integration decisions were taken and implemented through hardware and software deisgn choices:

- The novel payload is powered via a PWM-controlled relay, which can remove power from the payload
- The PWM-controlled relay is connected to a dedicated RC link, independent of the main flight control systems
- RC failsafe activation automatically switches the relay off
- A dedicated switch on the RC transmitter toggles the control output of the novel payload
- Activating (real) RTL for any reason will suppress the control output of the novel payload
- Due to implementation constraints (see more below…), GPS reception is always maintained by the UAV
- The novel payload does not command any changes to the altitude or airspeed of the UAV
- The novel payload does not output low-level attitude or rate commands

These measures, when correctly implemented and fully tested, ensure that the risk of loss of navigation control of the platform is no greater than any other flight, since the proven existing flight control architecture is effectively unchanged and can be completely isolated from the novel payload at any time through several manual and automatic means.

![Relay safety switch implementation](system_diagram_relay.png)
_Relay safety switch implementation_

## Final Hardware Solution

The companion computer is an Orange Pi Zero 2 SBC, used as a compromise between the Raspberry Pi 4 and Zero 2W. It has a lower power draw and footprint compared to the full size Pi, but still features ethernet and USB ports which the Pi Zero lacks. Running Armbian on the SBC showed some difficulty in using the GPIO UARTs, so I added a USB-TTL hub to allow both UARTs to be connected to the USB port and be accessed as USBs under /ttyUSBx. The LoRa air node is described above, and is connected to the hub on port 1, with port 0 used to connect to the spare SERIAL6 UART on the flight controller, which was setup as a simple MAVLink 2 interface.

On the ground, I setup my RC transmitter to output on both the internal and external RF modems, which allows for the control inputs to be sent simultaneously to the UAV default control on 868 MHz and also to the 2.4 GHz safety link which controls the payload power PWM relay. The RC failsafe was configured to turn off the PWM relay on lost link. The 433 MHz link was retained for GCS control and telemetry downlink.

The payload is powered from a custom pass-through 5V BEC, which isolates the power supply to the payload from the flight critical systems. If anything causes the BEC output to fail, power delivery to the rest of the UAV is unaffected.

![Hardware implementation block diagram](system_diagram_full.png)
_Hardware implememtation block diagram_

# Software Implementation

## RTH Algorithm Basics

My initial plan was to use the magnetic heading of the UAV, which is independent of GPS using the onboard magnetometer, and the LoRa-derived distance from home as the only two inputs to the navigation algorithm. The time taken between two distance measurements can inform the closing speed of the UAV to the home location, and the greater the closing speed the more accurately the nose of the aircraft is returning to home. By controlling the heading of the UAV, the aircraft can be turned left or right, and since a fixed-wing plane is always moving forwards the next closing speed measurement informs whether the turn has succeeded in increasing the closing speed. If not, turning in the opposite direction will guarantee we point the aircraft closer towards home.

This is a very basic and inelegant solution, but the aim is not to achieve a picture-perfect RTH manoeuvre - so long as the aircraft can be saved from a flyaway when GPS is lost and returned to VLOS range of the pilot, they can regain manual control and perform a landing. I mapped out the plan for the algorithm in a flowchart below.

![Initial algorithm flowchart](basic_algo_diagram.png)
_Initial algorithm flowchart_

## ArduPlane Current Support

ArduPilot currently supports a GNSS-denied beacon mode using multiple beacons for triangulation, with several significant caveats. It is only supported on Copter, requires dedicated third party hardware, and requires hardcoding the ground beacon position in the parameters.

Plane does not support these features, so a workaround is needed. Fortunately, ArduPilot systems use the MAVLink communication protocol as standard, and several libraries have been developed to make integrating external systems and sensors easy with existing flight controller hardware. I’m using the Python library - pymavlink - on the companion computer to communicate via serial with the flight controller on SERIAL6.

Another missing feature of ArduPlane is the ability to directly override heading when in Auto or Guided flight modes. I considered two methods to overcome this:

- Fly in Auto/Guided and continuously update the target waypoint with an arbitrary waypoint in the far distance extrapolated along the desired heading
- Fly in Cruise and continuously manipulate the roll/yaw command to change the heading (as the cruise controller takes RC yaw/roll commands and converts it to a heading command)

I opted for the former as it seemed simpler and doesn’t risk interfering with manual RC control. I should point out here that yes - GPS is *technically* being used in this solution, but it is not necessary at all! If ArduPlane natively allowed for heading control in Auto modes, that would work just as well and not require any GPS input. But due to the workaround requiring an extrapolated waypoint based on current and target locations, GPS data is used in this implementation. Sorry about that. I welcome any upset readers to implement heading control in an ArduPlane for me and I’ll be happy to test it for you!

## Pymavlink Basics

Pymavlink is a very accessible library with only a little Python knowledge required. A basic implementation to access a the VFR_HUD telemetry message and get the current airspeed is shown below: 

```
plane = mavutil.mavlink_connection('127.0.0.1:5761')
plane.wait_heartbeat()
while True:
    msg = plane.recv_match()
    if msg != None:
        msg = msg.to_dict()
            if msg['mavpackettype'] == 'VFR_HUD':
                airspeed = msg['airspeed']
```

It is also quite simple to request a specific message, which can save time waiting when a particular piece of data is needed. This is done using...............





## SITL Implementation

Setting up Arduplane software in the loop is as simple as cloning the repository and following the SITL guide in the documentation. By default, running `sim_vehicle.py` will open an instance of mavproxy which can be used for control and monitoring of the simulation as if flying in real life. The command line can be used to open additional ports, which I used to provide a connection to the navigation algorithm script. One thing to bear in mind is that some of the parameters on the vehicle need to be enabled or set correctly, such as FENCE_ENABLE, which I found out when trying to regression test the geofence breach response!



# References

[1] StuartsProjects, SX12XX-LoRa Library, GitHub - https://github.com/StuartsProjects/SX12XX-LoRa