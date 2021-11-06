# moist-controller
[![CC BY-SA 4.0][cc-by-sa-shield]][cc-by-sa]

## Overview

![Board Render](/images/Main_Board.png?raw=true)

The Moisture-Optimization through Solar Technology (MOiST) Controller is a distributed, solar-powered, ESP8266-based
controller that exposes standard irrigation valves into [Home Assistant](https://www.home-assistant.io/) through
the [ESPHome](https://esphome.io/) component.

The MOiST valve is designed to be:

### Simple

The controller does one thing: control a valve. Once added into Home Assistant, the controller exposeses a switch that
opens or closes the valve, sensors for water flow rate and total water consumption, and battery power monitoring. That's
it. You are free to use whatever mechanism you want to manage the valves, including HA automations, Node-RED flows, or
any other platform that integratges with Home Assistant.

### Flexible

This project started because we change up our gardens frequently, and we couldn't predict in advance where we would want
automatic irrigation, or how the yard would be zoned. The idea behind the controller is that you can mount your valve, 
controller, and solar panel on a stake, stick it in the ground where you need it, point it South (or North if you're in
the Southern Hemisphere), hook it up to a hose and be up and running quickly. Buried valves with hardwired power look
nicer, and would be more appropriate for a lawn, but for a food garden, this design provides more flexibility. The
controller should be able to drive any standard irrigation valve that uses a 9V DC latching solenoid.

### Self-contained

Running hoses is one thing, running power is another. The controller was designed to minimize power consumption to a
point where it can run off of a 5-Watt, 6-Volt solar panel and a 18650 battery with the amount of sun available during
growing season. Components were selected for low quiescent power, and the most inefficient parts of the controller are
powered off unless actively being used. On a quiet WiFi network, power consumption averages around 30mA

## Theory of Operation

![Schematic](/images/Main_Board.svg?raw=true)

The controller has 3 main functional areas: battery and power management, the solenoid driver, and the MCU.

### MCU (ESP8266)

The ESP8266 U5 is the heart of the system. It is running the ESPHome firmware, which handles the various outputs and sensors.
A bare ESP-WROOM-02D module was used rather than something like a D1 Mini, in order to remove unnecessary components and
minimize power requirements (USB-UART, LEDs, 5v regulator, etc.)

### Battery and Power Management

The system uses a standard 18560-style, unprotected Li-ion battery. Over-charge, over-discharge, over-current, and short
protection are provided by a BQ29733 protection IC U2, and solar charging is handled by a BQ24210 soalr charger U1 in battery
tracking mode, to maximize charging ability with low solar panel outputs. The charger can take any 6-volt solar panel, and
will charge at up to 800 mA. An LP3693MPX LDO regulator U3 provides 3.3V to the the rest of the components. An INA219 I2C
wattmeter U5 monnitors the battery voltage and current flowing in/out of the battery.

### Solenoid Driver

The solenoid driver consists of an unregulated boost converter PS1 that raises the 3V3 rail up to ~9V, a capacitor C1 to store
enough energy to drive the solenoid, an H-bridge driver U6 to drive the solenoid in the appropriate direction, and a comparator 
U7 with an 8.2V reference D2 to indicate when the capacitor is sufficiently charged. To open or close the solenoid, the MCU first
sets the DRV_EN pin HIGH, which powers up the boost converter and enables the H-Bridge. After a short delay for the comparator
to stabilize, the MCU waits for the capacitor voltage to exceed the 8.2v reference, causing the comparator to bring the
!CHGD line LOW. Once the capacitor is charged, the MCU pulses the DRV_1 or DRV_2 pin HIGH for the appropriate duration to open or
close the valve, respectively, and then brings DRV_EN low to power off the driver circuitry and preserve energy.

## A Note about WiFI:

ESP power consumption is significantly affected (sometimes by a factor of 2 or more) by how long the modem can stay in sleep mode.
To minimize power consumption, consider the following:

- Increase your DTIM beacon interval. This increases how long the ESP can stay asleep before waking up to check for data.
- Broadcast traffic causes the modem to wake up, including mDNS. Consider creating a separate WLAN and subnet for your low-power ESP devices, and use static IPs in ESPHome to minimize the amount of broadcast traffic reaching the ESPs.

## Current status and next steps

### 11-6-2021:
The breadboard prototype is functionally complete, and an order has been placed with OSHPark for a batch of boards for a
final prototype. 

### TODO:
- Test prototybe boards
- Create template YAML for ESPHome
- Model and print enclosure and mounting hardware
- Assemble valve components


This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg
