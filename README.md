Taperer-v0
==========

This repository contains software for an experimental fiber drawing/tapering set up built at the [FAMES](https://fames.indiana.edu/) lab. 


# Architecture

The tapering setup is a hybrid system consisting of a PLC controller that runs the tractors and a number of phidgets that control the rest of the system including various sensors, heating elements and pneumatic actuators.

# Software

The PLC component is implemented in IEC 61131-3 structured text via Parker's proprietary IDE (PAM 1.4), using PLCopen motion control function blocks. 
While it is separate and self contained, dumps of the [PAM project](PLC/taperer.projectarchive) and a functional motor controller [configuration](PLC/pd04_config.txt) are contained in this repository for reference. 

The main body of the software is implemented in Node-RED. It talks to the PLC component over OPC-UA and to phidgets via the node-red-contrib-phidget22 library through a phidget22networkserver instance running on the same machine. Graphical interface is implemented using the node-red-dashboard library. 

The dashboard provides the user with real time controls and visulalization of the most recent readings. 
Raw sensor readings are also pushed into an influxdb database for offline analysis. 

## Startup 

A [systemd service](Phidget/phidget.service) is used to start the phidget22networkserver. 
In the case of a fresh set up, this should be installed to the relevant path (i.e. /usr/lib/systemd/system/ or /etc/systemd/system/) and enabled with:

```
systemctl enable phidget
```

Node-RED and InfluxDB instances are run as Docker containers which will deploy at boot due to their `--restart always` policies.
They can be cold started by:

```
docker run -d --network 'host' -v nodered_data:/data --name nodered --restart always nodered/node-red
```

and 

```
docker run -d --network 'host' -v influxdb_data:/var/lib/influxdb --name influxdb --restart always influxdb
```

# Components 

## Tractors

Tractors are driven by a [PAC320-CWN21-3A](https://ph.parker.com/us/17607/en/pac-parker-automation-controller/pac320-cxn21-3a) controller and 4 sets of [PD-04C](https://ph.parker.com/us/en/pd-04c-single-axis-servo-drive-with-ethercat-interface-3-0a-1-100-230vac-1-1kva) servo drivers driving [PM-FALR5AM8N](https://www.parker.com/literature/Electromechanical%20North%20America/CATALOGS-BROCHURES/PSeries/PMotorSpecs_FAL40.pdf) servos with [PV60FN-100](http://www.parkermotion.com/products/Gearheads_and_Gearmotors__7102__30_32_80_567_29.html) gearheads (100:1) stacked onto [RX90-100-S2](https://www.parkermotion.com/products/Gearheads_and_Gearmotors__7065__30_32_80_567_29.html) gear boxes (100:1) yielding a 10000:1 effective gear ratio.

Arm 2 is geared into Arm 1 and Arm 3 is geared into Arm 4 in the PLC program (1:-1), meaning two sides of each tractor are synchronized at all times, but two tractors are controlled separately through 2 masters (Arm 1 and Arm 4). The primary mode of control is constant velocity continuous motion achieved by the MC_MoveVelocity function block from PLCopen. Conversion from engineering units to SI and vice versa is implemented on the Node-RED side. 

## Phidgets

The remainder of the system is interfaced by 22 phidgets connected into 4 VINT Hubs (HUB0000_0) connected into a 7-port USB hub (HUB0003_0) that is powered by the 24V DC supply in the cabinet (that also powers the PAC etc.). This hub will not do anything without external power, so the DC supply needs to be on for Phidgets to function. That means there is no way to independently power cycle the PLC or Phidget components in this set up.

10x TMP1100_0 Isolated Thermocouple Phidgets for thermocouples

1x REL1101_0 16x Isolated Solid State Relay Phidget for heating elements 

1x REL1100_0 4x Isolated Solid State Relay Phidget for solenoid valves

4x OUT1002_0 Isolated 16-bit Voltage Output Phidgets for transducers

2x DAQ_1500_0 Wheatstone Bridge phidgets for strain gauges

4x ENC1000_0 Quadrature Encoder Phidgets for the encoders


## Furnace

Furnace body consists of a copper tube with 10 axial through-bores for heating elements and a visual inspection cutout.
Copper tube is wholly shielded by a quartz tube outside, and partially shielded by two quartz tubes inside, which together with the geometry of the cutout achieves a temperature profile that is expected to peak at the center of the cylindrical volume.

Heating elements are [Watt-Flex cartridge heaters](https://daltonelectric.com/watt-flex-cartridge-heaters/cartridge-heaters) by Dalton Electric.
All 10 cartriges are of 1/4" diameter with dual grooves for external thermocouples.
6 of them run across the furnace (6" length, rated 400 Watts at 120 Volts, part no. G2C060) and the remaining 4 are shorter, and run in pairs due to being interrupted by the visual inspection cutout (2" length, rated 125 Watts at 120 Volts, part no. G2C020).
These are driven by mains power through 10 x AC solid state relays, which are in turn driven by 24V DC through a 16x solid state relay phidget (REL1101_0).

Thrmocouples are Type J (grounded), of 0.40" diameter with lengths 1/2" over their respective cartridges, and suppied by the same company.
Each thermocouple is read by an individual isolated thermocouple phidget (TMP1100_0).

## Pneumatics

Each tractor 

IP710-X60-D 2-60 psi
IP710-X100-D 2-100 psi

### Solenoids

V61R517A-A313JH


## Encoders

Positional encoders are magnetic and built with [RoLin](https://www.rls.si/eng/rolin-linear-and-rotary-incremental-magnetic-encoder-system) components by RLS.


Readheads (part no. RLM2HDA12BC15A00) are incremental type with flex connectors, 
terminated with RLACC connector adapters (part no. RLACC002), 
and eventually read by quadrature encoder phidgets (ENC1000_0).

Magnetic scales  (part no. MS05BM100BM080) hve a pole length of 2mm, which interpolated to 4096 steps at the readhead yields a maximum resolution of 0.488 micrometers. 

  * [RoLin datasheet](datasheets/RoLin_datasheet.pdf)
  * [RLACC datasheet](datasheets/RLACCD01_01.pdf)

## Strain gauges

