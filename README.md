# What Is This

This is the basic design and idea of an alternative to comma.ai's "car harness" for TSS-P Toyotas with a Driving Support ECU (DSU), this harness replaces the functionality of the SmartDSU, and allows the DSU to be "re-routed" to the camera part of the existing car harness, which is CAN2 on the panda. It allows the harness box to switch the DSU's output, enabling the filtration of CAN ID `0x343` on TSS-P Toyotas without the need for additional logical hardware, thus reducing the potential point of failure.

This will essentially act as an SmartDSU when implemented correctly (minus all the software issues). It will allow openpilot Longitudinal Control when your comma device is connected to the vehicle, whilst allowing the stock system to function normally when the comma device isn't being used.

# Schematic

A simple schematic of the harness has been attached for your convenience. I apologise for my dyslexic handwriting.

## Schematic and Wiring Explained

As it is nearly impossible to find a male DSU connector, it is suggested that you cut the wire on the vehicle's harness.

- Looking at the back (wire-side) of the DSU connector, counting from 1 from the right-top-most-PIN, you should sever the cable on PIN 10 and PIN 11;
- Tape the vehicle side of the severed wire to prevent a short circuit;
- Using a pair of 24-gauge speaker (or realistically, whatever you can find in the e-waste box) wire, extend the wire coming out of PIN 10 and PIN 11 from the DSU side of the harness up to the camera;
- Cut wire 5 and 11 on the camera side of the existing car harness;
- Connect PIN 10 of the DSU to PIN 5 of the camera harness, and PIN 11 to PIN 11 of the camera harness;
- Re-join the original camera harness back together;
- Zip tie, heat shrink, and insulate everything, and you should be done.

![Schematic of The DSU Re-Route Harness](schematic.jpeg?raw=true "Schematic")

# openpilot Software Change

A patch file is included in this repository for your reference.
