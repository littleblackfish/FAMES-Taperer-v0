Taperer-v0
==========

This repository contains software for an experimental fiber drawing/tapering set up built at the [FAMES](https://fames.indiana.edu/) lab. 


# Architecture

It is a hybrid system consisting of a PLC controller that runs the tractors and a number of phidgets that control the rest of the system including various sensors, heating elements and pneumatic actuators.

# Software

The PLC component is implemented in IEC 61131-3 structured text via Parker's proprietary IDE (PAM 1.4), using PLCopen motion control function blocks. 
While it is separate and self contained, a copy of the PAM project is contained in this repository for reference. 

The main software is implemented in Node-RED. It talks to the PLC component over OPC-UA and to phidgets via the node-red-contrib-phidget22 library through a phidget22networkserver instance running on the same machine. Graphical interface is implemented using the node-red-dashboard library. 

The dashboard provides the user with real time controls and visulalization of the most recent readings. 
Raw sensor readings are also pushed into an influxdb database for off-line analysis.   

# Hardware 

## PLC

Tractor component is self contained. It consist of a [PAC320-CWN21-3A](https://ph.parker.com/us/17607/en/pac-parker-automation-controller/pac320-cxn21-3a) controller and 4 sets of [PD-04C](https://ph.parker.com/us/en/pd-04c-single-axis-servo-drive-with-ethercat-interface-3-0a-1-100-230vac-1-1kva) servo drivers driving [PM-FALR5AM8N](https://www.parker.com/literature/Electromechanical%20North%20America/CATALOGS-BROCHURES/PSeries/PMotorSpecs_FAL40.pdf) servos with [PV60FN-100](http://www.parkermotion.com/products/Gearheads_and_Gearmotors__7102__30_32_80_567_29.html) gearheads (100:1) stacked onto [RX90-100-S2](https://www.parkermotion.com/products/Gearheads_and_Gearmotors__7065__30_32_80_567_29.html) gear boxes (100:1) yielding a 10000:1 effective gear ratio.

Arm 2 is geared into Arm 1 and Arm 3 is geared into Arm 4 in the PLC program (1:-1), meaning two sides of each tractor are synchronized at all times, but two tractors are controlled separately through 2 masters (Arm 1 and Arm 4). The primary mode of control is constant velocity continuous motion achieved by the MC_MoveVelocity function block from PLCopen. Conversion from engineering units to SI and vice versa is implemented on the Node-RED side. 

## Phidgets

The remainder of the system consist of 22 phidgets connected into 5 VINT Hubs (HUB0000_0) connected into a 7-port USB hub (HUB0003_0) that is powered by the 24V DC supply in the cabinet (that also powers the PAC etc.). This hub will not do anything without external power, so the DC supply needs to be on for Phidgets to function. That means there is no way to independently power cycle the PLC or Phidget components in this set up.

1x REL1100_0 4x Isolated Solid State Relay Phidget for solenoid valves

4x OUT1002_0 Isolated 16-bit Voltage Output Phidgets for transducers

2x DAQ_1500_0 Wheatstone Bridge phidgets for strain gauges

4x ENC1000_0 Quadrature Encoder Phidgets for the encoders

10x TMP1100_0 Isolated Thermocouple Phidgets for thermocouples

1x REL1101_0 16x Isolated Solid State Relay Phidget for heating elements 

## Furnace

## Pneumatics

## Encoders

## Strain gauges

