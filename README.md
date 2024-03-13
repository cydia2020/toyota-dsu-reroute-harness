# What Is This

This is the basic design and idea of an alternative to comma.ai's "car harness" for TSS-P Toyotas with a Driving Support ECU (DSU), this harness replaces the functionality of the SmartDSU, and allows the DSU to be "re-routed" to the camera part of the existing car harness, which is CAN2 on the panda. It allows the harness box to switch the DSU's output, enabling the filtration of CAN ID `0x343` on TSS-P Toyotas without the need for additional logical hardware, thus reducing the potential point of failure.

This will essentially act as an SmartDSU when implemented correctly (minus all the software issues). It will allow openpilot Longitudinal Control when your comma device is connected to the vehicle, whilst allowing the stock system to function normally when the comma device isn't being used.

# Schematic

A simple schematic of the harness has been attached for your convenience. I apologise for my dyslexic handwriting.

## Schematic and Wiring Explained

~~As it is nearly impossible to find a male DSU connector, it is suggested that you cut the wire on the vehicle's harness.~~

Plug and receptacle part number: DJ7324-0.7-11/21 (available on Taobao and eBay)

- Looking at the back (wire-side) of the DSU connector, counting from 1 from the right-top-most-PIN, you should sever the cable on PIN 10 and PIN 11;
- Tape the vehicle side of the severed wire to prevent a short circuit;
- Using around 2 to 3 meters of twisted pair wires with 120Ω impedance (or realistically, whatever you can find in the e-waste box), extend the wire coming out of PIN 10 and PIN 11 from the DSU side (the side of the OEM harness bundle with the female DSU connector) of the harness up to the camera;
- Cut wire 5 and 11 on the camera side of the existing comma car harness;
- Connect PIN 10 of the DSU to PIN 5 of the camera harness, and PIN 11 to PIN 11 of the camera harness;
- Re-join the original comma car harness back together;
- Zip tie, heat shrink, and insulate everything, modify openpilot according to the patch file (or use my fork) and you should be done.

![Schematic of The DSU Re-Route Harness](schematic.jpeg?raw=true "Schematic")

# Parts Needed (Plug-n-Play Option)

You will need the following connectors if you are making this as a PnP solution:
- DSU Plug and Receptacle: DJ7324-0.7-11/21
- comma.ai Toyota A Harness, which contains
 - Camera Plug (Female Camera Connector): 1318774-1
 - Camera Plug Female PIN Receptacles: 1123343-1 (*10)
 - Camera Receptacle (Male Camera Connector): 1565894-1
 - Camera Receptacle Male PIN Tabs: 1376109-1 (*10)


# FAQ

Here is a list of some questions that you may have regarding this harness. Feel free to reach out to me on Discord (cydia2020) should you have any other concerns about this project.

## Are You Planning To Sell It?
No, I do not have the time to fulfil orders, and I do not have access to the DSU's male connector. As a result, it is impossible for me to sell this harness. Consider making it yourself; it's really simple and it will be a great learning opportunity if you are just getting started with car mods and/or openpilot. Furthermore, if you have done the work correctly, any changes that you've made to the vehicle will be fully reversible when you decide to sell the car or stop using openpilot.

## What's The Difference Between This and a SmartDSU?
This is a lot cheaper (a few dollars for the cables compared to a few hundred for a panda), and it doesn't have any logic circuitry in it. Essentially, it replaces the SmartDSU with your comma device's built-in panda.

## If openpilot Is Not Enabled, or if The Comma Device Is Not Plugged-In, Would Stock ACC Still Function?
Yes.

## Is The Long CAN Bus Extension A Concern?
Maybe. Usually, CAN Bus cables shouldn't be extended this long without a termination resistor built-in to the furthest node. However, CAN Bus is very robust, and I haven't personally seen any issues with this setup.

## I am Getting "CAN Error: Check Connections" After Installing This Harness
First, check all connections.

openpilot needs some modification in order for this harness to work, as openpilot is expecting a few signals from bus0 of the panda, and now those signals have been moved to bus2. openpilot will think that the CAN Bus has a fault. An openpilot patch file that resolves this issue has been included in this repository for your reference.

## Are panda Code Modifications Required? Will This Get My Device Banned?
At this stage, panda code modifications are not required as panda filters `0x343` by default on all Toyota vehicles, and your device is unlikely to get banned.

## What Fork Supports This?
At this stage, only my fork supports it. However, support for this harness is extremely easy to implement and can be done within seconds. Send this repository to your fork's maintainer(s), and they will know what to do!
