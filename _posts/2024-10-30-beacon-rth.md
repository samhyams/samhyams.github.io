---
title: Beacon RTH - minimal SWaP-C GPS-denied navigation
date: 2024-10-30 16:30:06 +0000
categories: [Projects, Pinegrove]
tags: [uas, uav, ardupilot, arduplane, pymavlink, gps, gnss, denied]     # TAG names should always be lowercase
description: GPS-denied RTH for MAVLink systems using LoRa radio ranging
toc: false
media_subpath: /beacon-rth/
image:
  path: DSC02430.jpg
  alt: Hardware implementation in the Orca fuselage 
---

## Introduction

UAVs commonly use return-to-home (RTH) as a failsafe for situations where the aircraft should return home autonomously, such as a loss of comms or low battery scenario. RTH typically relies on GPS position navigation, with dead-reckoning using compass heading as a backup when GPS position is unavailable. This dead-reckoning mode is not always reliable, as it relies on accurate wind estimation to account for drift during the return flight, which can cause significant position errors as distance to home increases.

Emerging novel methods of improving GPS-denied navigation include optical flow and computer vision with a stored topology database to triangulate position. These methods are complex and can require significant weight and power overheads on the aircraft, which impacts endurance and cost. These methods can also be more difficult to perform on fixed-wing aircraft which must always be moving across the terrain.

LoRa radios offer a ranging feature, where the point-to-point distance between two radio modems can be determined using time-of-flight. These radios are very low SWaP-C and therefore have good potential for use as a beacon system. The RF properties are also relatively benign, as they can be used in ISM bands and at low EIRP to prevent EM interoperability issues with other systems on the aircraft. In addition, they are claimed to be able to function below the noise floor which improves resilience against jamming and benefits ultimate range.

By knowing the distance from home, the relative closing speed of the aircraft to the home location can be calculated and compared to previous results to determine the trajectory of the aircraft relative to the home location over time. This can ultimately be used to crudely guide the aircraft back towards the home position. This project doesn't aim to match the accuracy of GPS-guided RTH or even the other GPS-denied solutions mentioned above, but the aim is to autonomously bring the UAV back in the direction of home to a point close enough that the pilot can gain visual, retake control, and land - all without the need for any GNSS-derived input data.

As this solution only requires a single ground-based radio beacon, I refer to it as Beacon RTH.

## Investigating LoRa ranging performance

Inspiration for this project came from reading through the Semtech LoRa documentation whilst working on another project. The documentation describes the in-built radio wave time-of-flight functionality, which can determine the distance between to LoRa radio modems.

An open source Arduino library for LoRa radio modules, shared by StuartsProjects on GitHub [1], not only offers an easy way to get communicating between LoRa modems, but also a ready made ranging example which can be adapted for custom use. The modifications needed were very minor; I set one ground node as a ‘transmitter’ to broadcast ranging requests and one airborne node as a ‘receiver’ to respond to the ranging request. The ground node then output the resulting distance measurement via serial to a connected laptop where it could be recorded.

![Hardware implemetation of the LoRa air node](air_node_front.jpg)
_Perfboard setup of the LoRa air node_

The radio modems consist of Ebyte E28 SX1280 modems mounted on breakout PCBs to allow breadboarding. The air node fits on a 6x4 cm perfboard with an Arduino Pro Micro and a cheap BEC to provide 3.3 V to the circuitry, keeping the air node self-contained when attached to a power source. The ground node is less permanent, with the same components mounted on a breadboard to be powered from an external power source or USB port.

![Air node testing](field_setup.jpg)
_Air node attached to a tripod with structural tape for initial testing_

I setup the air node to broadcast from my Orca aircraft and placed the ground node next to my GCS. After flying patterns and orbits for about 20 minutes, I analysed the data to compare the LoRa ranging measurements against the calculated GPS distance from the aircraft, and you can see the results below.

![LoRa ranging test results](distance_comparison_sub.png)
_Comparison of GPS and LoRa ranging distance measurements during flight_

The results were very promising with only a small delta between the LoRa and GPS results, which can easily be accounted for by several factors including GPS accuracy and the aircraft home position vs GCS position offset of a few metres. For the Beacon RTH application, the accuracy shown is more than good enough as the ranging results can clearly be relied on to determine if the UAV is moving away from or towards the home point.

## System Setup

My initial ranging tests took place over a year ago, and I didn’t start my work on the larger RTH solution until autumn this year. Thankfully, I’d documented my previous work well enough to give myself a head start. The first task is determining how to setup the system for navigation control in a way which is both performant and safe. 

As mentioned before, the LoRa radio modems are set up for ranging using Arduino microcontrollers to provide a serial interface. I switched the roles of the air and ground node, so the air node requests a ranging measurement and outputs the response from the ground node on its serial port. This provides automatic ranging updates at approximately 2 Hz for use by other systems on the UAV. The airborne LoRa node onboard the UAV is connected to a companion computer. The companion computer is also connected by UART to a serial port on the flight controller to provide bidirectional communications, allowing the companion computer to both receive flight data and send control commands. The ground LoRa node is a standalone modem and microcontroller which responds to the ranging requests made by the airborne node.

![Basic System Diagram](system_diagram_basic.png)
_Basic system overview_

## Hardware Implementation

The project is flown aboard my *Orca*, which I built over a year ago for the purpose of flying custom payloads and other projects. The large payload bay is perfect for this project, and I added some SMA outlets on the sides of the fuselage to allow for easy mounting of the payload antennas. The platform uses a Matek F405 V2 flight controller running ArduPlane 4.5, with 868 and 433 MHz control and telemetry links leaving the S band free for project use. Main battery voltage is 16.8 V nominal from a 4S LiPo battery.

### System Safety

Writing a bunch of performance requirements leans a little too close to my day job, but given this project involves handing over navigation control to some of my own hastily-written hacked together code, I think it’s best practice to stipulate a few key safety requirements. Here are three very high level requirements which must be met by the system as a whole:

- RC commands shall always override the novel payload
- UAV failsafes shall always override the novel payload
- The novel payload shall not command the UAV to make an unsafe manoeuvre

To meet these demands, several key integration decisions were taken and implemented through hardware and software deisgn choices:

- The novel payload is powered via a PWM-controlled relay, which can remove power from the payload
- The PWM-controlled relay is connected to a dedicated RC link, independent of the main flight control systems
- RC failsafe activation automatically switches the relay off
- A dedicated switch on the RC transmitter toggles the control output of the novel payload
- Activating (real) RTH for any reason will suppress the control output of the novel payload
- Due to implementation constraints (see more below…), GPS reception is always maintained by the UAV
- The novel payload does not command any changes to the altitude or airspeed of the UAV
- The novel payload does not output low-level attitude or rate commands

These measures, when correctly implemented and fully tested, ensure that the risk of loss of navigation control of the platform is no greater than any other flight, since the proven existing flight control architecture is effectively unchanged and can be completely isolated from the novel payload at any time through redundant manual and automatic means.

![Relay safety switch implementation](system_diagram_relay.png)
_Relay safety switch implementation_

### Final Hardware Solution

The companion computer is an Orange Pi Zero 2 SBC, used as a compromise between the Raspberry Pi 4 and Zero 2W. It has a lower power draw and footprint compared to the full size Pi, but still features ethernet and USB ports which the Pi Zero lacks. Running Armbian on the SBC showed some difficulty in using the GPIO UARTs, so I added a USB-TTL hub to allow both UARTs to be connected to the USB port and be accessed as USBs under /ttyUSBx. The LoRa air node is described above, and is connected to the hub on port 1, with port 0 used to connect to the spare SERIAL6 UART on the flight controller, which was setup as a  MAVLink 2 interface.

On the ground, I setup my RC transmitter to output on both the internal and external RF modems, which allows for the control inputs to be sent simultaneously to the UAV default control on 868 MHz and also to the 2.4 GHz safety link which controls the payload power PWM relay. The RC failsafe was configured to turn off the PWM relay on lost link. The 433 MHz link was retained for GCS control and telemetry downlink.

The payload is powered from a custom pass-through 5V BEC, which isolates the power supply to the payload from the flight critical systems. If anything causes the BEC output to fail, power delivery to the rest of the UAV is unaffected.

![Hardware implementation block diagram](system_diagram_full.png)
_Hardware implementation block diagram_

## Software Implementation

### RTH Algorithm Basics

My initial plan was to use the magnetic heading of the UAV, which is independent of GPS using the onboard magnetometer, and the LoRa-derived distance from home as the only two inputs to the navigation algorithm. The time taken between two distance measurements can inform the closing speed of the UAV to the home location, and the greater the closing speed the more accurately the nose of the aircraft is returning to home. By controlling the heading of the UAV, the aircraft can be turned left or right, and since a fixed-wing plane is always moving forwards the next closing speed measurement informs whether the turn has succeeded in increasing the closing speed. If not, turning in the opposite direction will guarantee we point the aircraft closer towards home.

This is a very basic and inelegant solution, but the aim is not to achieve a picture-perfect RTH manoeuvre - so long as the aircraft can be saved from a flyaway when GPS is lost and returned to VLOS range of the pilot, they can regain manual control and perform a landing. I mapped out the plan for the algorithm in a flowchart below.

![Initial algorithm flowchart](basic_algo_diagram.png)
_Initial algorithm flowchart_

### ArduPlane Current Support

ArduPilot currently supports a GNSS-denied beacon mode using multiple beacons for triangulation, with several significant caveats. It is only supported on Copter, requires dedicated third party hardware, and requires hardcoding the ground beacon position in the parameters.

Plane does not support these features, so a workaround is needed. Fortunately, ArduPilot systems use the MAVLink communication protocol as standard, and several libraries have been developed to make integrating external systems and sensors easy with existing flight controller hardware. I’m using the Python library - pymavlink - on the companion computer to communicate via serial with the flight controller on SERIAL6.

Another missing feature of ArduPlane is the ability to directly override heading when in Auto or Guided flight modes. I considered two methods to overcome this:

- Fly in Auto/Guided and continuously update the target waypoint with an arbitrary waypoint in the far distance extrapolated along the desired heading
- Fly in Cruise and continuously manipulate the roll/yaw command to change the heading (as the cruise controller takes RC yaw/roll commands and converts it to a heading command)

I opted for the former as it seemed simpler and doesn’t risk directly interfering with manual RC control inputs. I should point out here that yes - GPS is *technically* being used in this solution, but it is not necessary at all! If ArduPlane natively allowed for heading control in Auto modes, that would work just as well and not require any GPS input. But due to the workaround requiring an extrapolated waypoint based on current and target locations, GPS data is used in this implementation. Sorry about that. I welcome any upset readers to implement heading control in an ArduPlane for me and I’ll be happy to test it for you!

### Pymavlink Basics

Pymavlink is a very accessible library with only a little Python knowledge required, and there are plenty of online resources which can provide a headstart in understanding how to use it [2,3]. As an example, basic implementation to access the VFR_HUD telemetry message and get the current airspeed is shown below: 

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

It is also quite simple to request a specific message, which can save time waiting when a particular piece of data is needed. This is done using a `command_long` message sent to the MAV, where the command is to request a message and the requested message type is defined in `param1`. We then wait until the requested message type is seen in the inbound MAVLink stream and output the message. The example below shows an implementation for requesting the MAV home position:

```
# Create request message command
    request_message_command = dialect.MAVLink_command_long_message( 
        target_system=plane.target_system,
        target_component=plane.target_component,
        command=dialect.MAV_CMD_REQUEST_MESSAGE,
        confirmation=0,
        param1=dialect.MAVLINK_MSG_ID_HOME_POSITION,
        param2=0,
        param3=0,
        param4=0,
        param5=0,
        param6=0,
        param7=0
      )
    # Send command to the vehicle
    plane.mav.send(request_message_command)
    # Get response
    message = plane.recv_match(type=dialect.MAVLink_home_position_message.msgname,
                                 blocking=True).to_dict()
```

By changing `param1` to `MAVLINK_MSG_ID_<x>` and the type in `recv_match()` to `MAVLink_<x>_message`, it is possible to request any MAVLink message type that your firmware can report.

As for sending commands to the MAV, the same procedure is used but the command and params within the `command_long` message are changed. As an example for changing the flight mode, we send the `DO_SET_MODE` command and define the numeric representation of the mode as `param2`. The MAV will send a `COMMAND_ACK` message after it has processed our request, so we wait to check the result of this at the end:

```
# Create change mode message
    set_mode_message = dialect.MAVLink_command_long_message(
        target_system=plane.target_system,
        target_component=plane.target_component,
        command=dialect.MAV_CMD_DO_SET_MODE,
        confirmation=0,
        param1=dialect.MAV_MODE_FLAG_CUSTOM_MODE_ENABLED,
        param2=<x>,
        param3=0,
        param4=0,
        param5=0,
        param6=0,
        param7=0
    )
    # Change flight mode
    plane.mav.send(set_mode_message)
    # Wait for response
    while True:
        # Catch COMMAND_ACK message
        message = plane.recv_match(type=dialect.MAVLink_command_ack_message.msgname, 
                                   blocking=True).to_dict()
        # Check the COMMAND_ACK is for DO_SET_MODE
        if message["command"] == dialect.MAV_CMD_DO_SET_MODE:
            # Check the command is accepted or not
            if message["result"] == dialect.MAV_RESULT_ACCEPTED:
                # Inform the user
                print('Flight mode set')
            # Not accepted
            else:
                # Inform the user
                print('Failed to set flight mode')
            # Break the loop
            return
```

### Other Pitfalls

I found a lot of issues running the scripts resulted from using the python `time.sleep()` function, as it seemed to cause latency in reading the incoming serial streams. I replaced all usages of `time.sleep()` with a custom pause duration function which seemed to help performance.

### SITL Implementation

Setting up ArduPlane software in the loop is as simple as cloning the repository and following the SITL guide in the documentation. By default, running `sim_vehicle.py` will open an instance of GCS utility Mavproxy which can be used for control and monitoring of the simulation as if flying in real life. The command line can be used to open additional ports, which I used to provide a connection to the navigation algorithm script. One thing to bear in mind is that some of the parameters on the vehicle need to be enabled or set correctly, such as FENCE_ENABLE, which I found out when trying to regression test the geofence breach response!

### Navigation Algorithm Development

My initial steps with the navigation algorithm started with getting target location commands working. Getting current position data from MAVLink GLOBAL_POSITION_INT messages was straightforward, as was using location and heading to determine the target location from extrapolation, but trying to continuously update the target location in AUTO proved tricky. I tried several iterations of code using snippets from both [2] and [3] but ultimately settled on a simpler implementation when in GUIDED flight mode, which performs as expected. The target waypoint is extrapolated 1 km ahead of the current location as this point will never be reached during normal operation of the algorithm in real life. 

With a distance to home function added, dynamic navigation based on distance to home could be developed. Initially I added the ability for the aircraft to turn right, which worked well, then I added logic to also turn left, and to switch turn direction if the previous turn moved the aircraft further from home. Finally, I included logic to adjust the magnitude of the next turn (the delta between current and next commanded heading) based on how close the closing speed is to the airspeed setpoint - assuming no wind, the maximum closing speed equals the airspeed setpoint. Effectively, this reduces the size of turn commanded as the accuracy of the aircraft's direction towards home increases.

This all worked remarkably well considering the crude nature of the algorithm, so I next added a custom wind correction function to allow for operation in wind without GPS-derived wind estimation. The function takes the original windspeed and direction estimation determined by ArduPlane at initialisation of the script (i.e. when GPS is lost by the platform) and calculates the corrected ground speed and ground course based on the current UAV magnetic heading.

When the script is started, the UAV home position and last known wind measurements are used to initialise the navigation controller. The airspeed setpoint, distance to home, and magnetic heading are all that is required for the controller to function, and my testing proves that it always returns the UAV to the vicinity of home, regardless of initial heading and distance to home. Indeed, if low or zero wind speeds are present, the UAV will often end up orbiting or performing a figure of eight pattern around the home point once reached! The plot below shows that the estimated wind direction remains within 4 degrees and estimated groundspeed remains within 3 m/s for all aircraft headings, which is sufficient for this application.

![Synthetic wind approximation delta](wind_plot.png)
_Comparison of custom estimated wind components with actual, against aircraft heading. Wind direction shown by black arrow_

The video below shows a brief summary of the performance of the navigation controller in the simulator.

{% include embed/youtube.html id='y_hJwu-_NvM' %}

Before moving on to hardware testing, I regression tested the operation of the geofence failsafes, which is also shown in the summary video. The inclusion of a 'RTL check' in the script, to prevent setting a new flight mode if the current flight mode is RTL, ensures that the ArduPlane stock RTL action is not overridden at any point unless commanded by the pilot and was shown to function as expected during SITL testing.

## Hardware Testing

Some issues with the SITL code cropped up when transitioning over to the hardware. One key issue was my inability to get the MAVLink message HIGH_LATENCY2 working on hardware - despite following the instructions for enabling it after boot via Mission Planner. This was unfortunate as HIGH_LATENCY2 is the only default MAVLink message which provides airspeed setpoint by default, so I had to refactor the script to request the airspeed error and use this to calculate the airspeed setpoint from the the error and measured airspeed. I later found that the HIGH_LATENCY2 feature is not enabled by default on pre-compiled ArduPilot firmwares for 1 MB flash boards, which explains the lack of functionality.

Adding in a master switch was straightforward. Channel 8 from the RC transmitter is read from the flight controller and the script is only actioned if the switch is in the down position. I adjusted the transmitter output so that switch up position (1000 us PWM) switches off the relay, switch middle position (~1500 us PWM) turns on the relay but does not activate the script, and down position (~2000 us PWM) activates the navigation script. The script is also setup using a systemd service to automatically launch at bootup, so the script can be effectively reset during flight by power cycling the relay.

![Hardware Install](DSC02430.jpg)
_Hardware installation in payload bay_

One additional issue which required troubleshooting was the ability of Python to read two hardware UART serial streams simultaneously. Trying to run open both the MAVLink and Arduino serial streams in the script caused significant latency in incoming data packets, on the order of 10 seconds or more. Whilst the navigation script does not require millisecond-accurate data due to the crude implementation, latency of over a second is not at all sufficient. I tried a few potential solutions to fix this, including the Python multithreading/multiprocessing libraries and simply opening and closing the UART ports as needed, but neither gave a satisfactory result. Instead, a suitable workaround was to run the Arduino script separately in the background and dump the LoRa distance measurement to a temporary .txt file, which can then be read asynchronously by the navigation script which has the MAVLink UART port open. Although not elegant, this solution solved the latency issues and works well enough for this proof of concept.

## Flight Testing

Initial flight testing was a mixed bag, with a lot of minor issues which needed ironing out, including some of those mentioned above. I certainly underestimated the difficulty of needing to single-handedly fly the aircraft and monitor the GCS whilst anticipating what the navigation script would do. The script would frequently hang during operation, and troubleshooting was made all the more difficult by the absence of any downlink from the SBC to the ground, so if anything went wrong with the script I was in the dark as to exactly what.

![Flight test setup](DSC02423.jpg)
_Set up in the field for flight testing_

I added some detailed logging to the script to record where and when the script hung, which finally allowed some minor success in seeing the navigation algorithm function in real life. After a few mid-air reboots, the script started to respond and turn the aircraft towards home:

![Initial response](beacon_hitl_gps0.png)
_Flight test initial navigation script response. Black arrow shows direction of flight_

After a few turn commands, the script stopped requesting further turns for an unknown reason, but it was encouraging to see some proof of success, no matter how small! I made some more minor adjustments to the script setup, and achieved some more convincing results:

![Long duration response](beacon_hitl_gps1.png)
_Flight test navigation response extended. Black arrow shows direction of flight_

These initial attempts were made using a distance to home calculated from the GPS coordinates, as this was used in SITL testing, but - as described above - the navigation algorithm should perform identically irrespective of the home distance source. Clearly, the script is functioning as expected as the aircraft 'orbits' the home point with the switches between turning left and right obvious, but the script still hangs at some point, after which the aircraft flies along the last commanded heading. One final test was performed with the LoRa ranging measurement active for the distance to home measurement:

![LoRa ranging flight test response](beacon_hitl_lora0.png)
_Flight test response with LoRa ranging enabled. Black arrow shows direction of flight_

Clearly not the expected result, showing a circular loiter being pushed downwind, but the cause was simple - the ground end of the LoRa link had turned off since the power draw was so low that the power supply had deactivated its output! This results in a measured home distance of 0 metres on the aircraft, and backtesting in SITL with a fixed 0 m distance to home input shows the same orbit-in-place behaviour as observed in the flight test.

There are still some issues with the automatic startup of the scripts on the SBC, which makes testing difficult. Although not fully successful yet, I wanted to write up the project results so far to record the development, pitfalls, and outcomes so that the future blog post isn't quite so long. More to come once the last few issues are solved!

## References

[1] StuartsProjects, SX12XX-LoRa Library, GitHub - https://github.com/StuartsProjects/SX12XX-LoRa

[2] khancyr, Pymavlink complete example for copter control, GitHub - https://github.com/ArduPilot/pymavlink/pull/503

[3] mustafa-gokce, Ardupilot Software Development - Pymavlink, GitHub - https://github.com/mustafa-gokce/ardupilot-software-development/tree/main/pymavlink
