# Project Pinegrove

Beacon RTH for UAS in GNSS-denied environments using LoRa radio ranging

### Basics

UAVs commonly use return-to-home (RTH) as a failsafe for situations where the aircraft should return home autonomously, such as a loss of comms or low battery scenario. RTH typically relies on GPS position navigation, with dead-reckoning using compass heading as a backup when GNSS position is unavailable. This dead-reckoning mode is not always reliable, as it relies on accurate wind estimation to account for drift during the return flight, which can cause significant position errors as distance to home increases.

Emerging novel methods of improving GNSS-denied navigation include optical flow and computer vision with a stored topology database to triangulate position. These methods are complex and can require significant weight and power overheads on the aircraft, which impacts endurance and cost. These methods can also be more difficult to perform on fixed-wing aircraft which must always be moving across the terrain.

LoRa radios offer a ranging feature, where the point-to-point distance between two radio modems can be determined using time-of-flight calculations. These radios are very low SWaP-C and therefore have good potential for use as a beacon system. The RF properties are also relatively benign, as they can be used in ISM bands and at low powers to prevent significant EM issues with other systems on the aircraft. In addition, they are claimed to be able to function below the noise floor which improves resilience against jamming.

### LoRa Ranging

Inspiration for this project came from reading through the Semtech LoRa documentation whilst working on another project. The documentation describes the in-built radio wave time-of-flight functionality, which can 

An open source arduino library for LoRa radio modules, shared by StuartsProjects on GitHub[1], not only offers an easy way to get communicating between LoRa modems, but also a ready made ranging example (and some testing results!) which can be adapted for custom use. The modifications needed were very minor; I set one ground node as a ‘transmitter’ to broadcast ranging requests and one airborne node as a ‘receiver’ to respond to the ranging request. The ground node then output the distance via serial to a connected laptop where it could be recorded.

<images of air node>

I setup the air node to broadcast from my Orca aircraft and placed the ground node next to my GCS. After flying patterns and orbits for about 20 minutes, I analysed the data to compare the LoRa ranging measurements against the calculated GPS distance from the aircraft, and you can see the results below.

<plots of accuracy>

The results were very promising, only a small delta between the LoRa and GPS results, which can easily be accounted for by several factors (accuracy of GPS, home position vs GCS position offset of a metre or two). For the RTH application, the accuracy shown is more than good enough as the ranging results can clearly be relied on to determine if the UAV is moving away from or towards the home point.

### System Setup

My initial ranging tests took place over a year ago, and I didn’t start my work on the larger RTH solution until summer turned to autumn this year. Thankfully, I’d documented my previous work well enough to give myself a head start. FIrst off is determining how to setup the system in a way which is both performant and safe. 

As mentioned before, the LoRa radio modems are set up for ranging using arduino microcontrollers to provide a serial interface. This provides automatic ranging updates at 1 Hz and outputs distance over the serial port for use by other systems. The airborne LoRa node onboard the UAV is connected to a companion computer. The companion computer is also connected by UART to a serial port on the flight controller to provide bidirectional communications, allowing the companion computer to both receive flight data and send control commands. The ground LoRa node is a standalone modem and microcontroller which responds to the ranging requests made by the airborne node.

![system_diagram_basic.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/77c360b0-8679-4b1d-827d-8a9699f5d205/aee86bde-c6db-4faf-83ce-43432364cdea/system_diagram_basic.png)

### System Safety

Writing a bunch of requirements leans a little too close to my day job, but given this project involves handing over navigation control to some of my own hastily-written hacked together code, I think it’s best practice to stipulate a few key safety requirements. Here are two very high level requirements which must be met by the system as a whole:

- RC commands shall always override the novel payload
- Failsafes shall always override the novel payload
- The novel payload shall not command the UAV to make an unsafe manoeuvre

To meet these demands, several key integration decisions were taken and implemented through hardware and software deisgn choices:

- The novel payload is powered via a PWM-controlled relay, which can remove power from the payload
- The PWM-controlled relay is connected to a dedicated RC link, independent of the main flight control systems
- RC failsafe activation automatically switches the relay off
- A dedicated switch on the RC transmitter toggles the control output of the novel payload
- Activating (real) RTL for any reason will suppress the control output of the novel payload
- Due to implementation constraints (see more below…), GPS reception is always maintained by the UAV
- The novel payload does not command any changes to the altitude or airspeed of the UAV

<image of system architecture>

These measures, when correctly implemented and fully tested, ensure that the risk of loss of navigation control of the platform is no greater than any other flight, since the proven existing flight control architecture is effectively unchanged and can be completely isolated from the novel payload at any time through several manual and automatic means.

### Basic Algorithm Plan

My initial plan was to use the known magnetic heading of the UAV, which is independent of GPS provided you have an onboard magnetometer, and the LoRa-derived distance from home as the only two inputs to the navigation algorithm. The time taken between two distance measurements can inform the closing speed of the UAV to the home point, and the greater the closing speed the more accurately the nose of the aircraft is pointing towards home. By controlling the heading of the UAV, the aircraft can be turned left or right, and since a fixed-wing plane is always moving forwards the next closing speed measurement informs whether the turn has succeeded in increasing the closing speed. If not, turning in the opposite direction will guarantee we point the aircraft closer towards home.

This is a very basic and inelegant solution, but the aim is not to achieve a picture-perfect RTH manoeuvre - so long as the aircraft can be saved from a flyaway when GPS is lost and returned to VLOS range of the pilot, they can regain manual control and perform a landing.

### Ardupilot Current Support

ArduPilot currently supports a GNSS-denied beacon mode using multiple beacons for triangulation - with several significant caveats. It is only supported on Copter, requires dedicated third party hardware, and requires hardcoding the ground beacon position in the parameters.

Plane does not support these features, so a workaround is needed. Fortunately, ArduPilot systems use the MAVLink communication protocol as standard, and several libraries have been developed to make integrating external systems and sensors easy with existing flight controller hardware. I’m using the Python library - pymavlink - on the companion computer to communicate via serial with the flight controller on a spare UART port.

Another missing feature of ArduPlane is the ability to directly override heading when in Auto or Guided flight modes. I considered two methods to overcome this:

- Fly in Auto and continuously update the target waypoint with an arbitrary  waypoint in the far distance extrapolated along the desired heading
- Fly in Cruise and continuously manipulate the roll/yaw command to change the heading (as the cruise controller takes RC yaw/roll commands and converts it to a heading command)

I opted for the former as it seemed simpler and doesn’t risk interfering with manual RC control. I should point out here that yes - GPS is *technically* being used in this solution, but it is not necessary at all! If Ardupilot natively allowed for heading control in Auto modes, that would work just as well and not require any GPS input. But due to the workaround requiring an extrapolated waypoint based on current and target locations, GPS is used in this implementation. Sorry about that.

I welcome any upset readers to implement heading control in an ArduPlane for me and I’ll be happy to test it for you!

### Software Implementation

### Hardware Implementation

### References

[1] StuartsProjects, SX12XX-LoRa Library, GitHub - https://github.com/StuartsProjects/SX12XX-LoRa