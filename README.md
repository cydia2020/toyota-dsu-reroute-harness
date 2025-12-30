# What Is This

This is the basic design and idea of an alternative to comma.ai's "car harness" for TSS-P Toyotas with a Driving Support ECU (DSU), this harness replaces the functionality of the SmartDSU, and allows the DSU to be "re-routed" to the camera part of the existing car harness, which is CAN2 on the panda. It allows the harness box to switch the DSU's output, enabling the filtration of CAN ID `0x343` on TSS-P Toyotas without the need for additional logical hardware, thus reducing the potential point of failure.

This will essentially act as an SmartDSU when implemented correctly (minus all the software issues). It will allow openpilot Longitudinal Control when your comma device is connected to the vehicle, whilst allowing the stock system to function normally when the comma device isn't being used.

# Schematic

A simple schematic of the harness has been attached for your convenience. I apologise for my dyslexic handwriting.

## Schematic and Wiring Explained

- Looking at the back (wire-side) of the DSU connector, counting from 1 from the right-top-most-PIN, you should sever the cable on PIN 10 and PIN 11;
- Tape the vehicle side of the severed wire to prevent a short circuit;
- Using around 2 to 3 meters of twisted pair wires with 120Ω impedance (or realistically, whatever you can find in the e-waste box), extend the wire coming out of PIN 10 and PIN 11 from the DSU side (the side of the OEM harness bundle with the female DSU connector) of the harness up to the camera;
- Cut wire 5 and 11 on the camera side of the existing comma car harness;
- Connect PIN 10 of the DSU to PIN 5 of the camera harness, and PIN 11 to PIN 11 of the camera harness;
- Re-join the original comma car harness back together;
- Zip tie, heat shrink, and insulate everything, modify openpilot according to the patch file (or use my fork) and you should be done.

![Schematic of The DSU Re-Route Harness](/docs/schematic.jpeg?raw=true "Schematic")

# Parts Needed (Plug-n-Play Option)

You will need the following connectors if you are making this as a PnP solution:
- DSU Plug and Receptacle: DJ7324-0.7-11/21
- comma.ai Toyota A Harness, or, you can make your own harness by purchasing:
  - Camera Plug (Female Camera Connector): 1318774-1
  - Camera Plug Female PIN Receptacles: 1123343-1 (*10)
  - Camera Receptacle (Male Camera Connector): 1565894-1
  - Camera Receptacle Male PIN Tabs: 1376109-1 (*10)

| | |
|:-------------------------:|:-------------------------:|
|<img width="1604" alt="DSU Side of the Harness" src="/docs/dsu_male_female.jpg">  DSU Side of the Harness |  <img width="1604" alt="DSU Female Side PIN Assignment" src="/docs/dsu_female_pinout.jpg"> DSU Side Female PIN Assignment|
|<img width="1604" alt="DSU Male PIN To Be Cut" src="/docs/dsu_male_pinout.jpg">  DSU Male Side PIN To Be Cut|  <img width="1604" alt="Camera Female Side PIN Assignment" src="/docs/camera_pinout.jpg"> Camera Female Side PIN Assignment|


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

openpilot needs some modification in order for this harness to work, as openpilot is expecting a few signals from bus0 of the panda, and now those signals have been moved to bus2. openpilot will think that the CAN Bus has a fault. 

These information below will help you modify opendbc.

```diff
# --- This section is to modify car/toyota/carstate.py ---
diff --git a/selfdrive/car/toyota/carstate.py b/selfdrive/car/toyota/carstate.py
index e4ea0d30f..e9284f983 100644
--- a/selfdrive/car/toyota/carstate.py
+++ b/selfdrive/car/toyota/carstate.py
@@ -128,9 +128,11 @@ class CarState(CarStateBase):
       conversion_factor = CV.KPH_TO_MS if is_metric else CV.MPH_TO_MS
       ret.cruiseState.speedCluster = cluster_set_speed * conversion_factor
 
# cp_acc is Toyota car state's variable that records which CAN Bus the ACC message (0x343) is on
# cp_cam is the panda's camera CAN Bus, and cp is the panda's car CAN Bus

# in short, the stock op logic is - on TSS2.0 cars excluding those that control ACC with the RADAR,
# the camera does longitudinal control, so cp_cam = TSS2.0 minus radar acc cars

# because we are re-routing DSU to the camera CAN Bus, the stock long messages have been moved to the panda's camera bus, 
# so we modify this logic to include TSS-P cars that has flag DSU_BYPASS
-    cp_acc = cp_cam if self.CP.carFingerprint in (TSS2_CAR - RADAR_ACC_CAR) else cp
+    cp_acc = cp_cam if (self.CP.carFingerprint in (TSS2_CAR - RADAR_ACC_CAR) or \
+             bool(self.CP.flags & ToyotaFlags.DSU_BYPASS.value)) else cp


# Here, because openpilot listens for various signals from 'ACC_CONTROL' and 'PCS_HUD' for pass-thru and to facilitate op alerts,
# and on TSS2 and DSU re-routed cars, ACC_CONTROL is on cp_cam, so we need to include DSU_BYPASS flag in the condition to allow
# op to listen to the messages that it needs from the correct CAN Bus (in our re-routed case, the camera bus)
-    if self.CP.carFingerprint in TSS2_CAR and not self.CP.flags & ToyotaFlags.DISABLE_RADAR.value:
+    if (self.CP.flags & ToyotaFlags.DSU_BYPASS.value) or (self.CP.carFingerprint in TSS2_CAR \
+       and not self.CP.flags & ToyotaFlags.DISABLE_RADAR.value):
       if not (self.CP.flags & ToyotaFlags.SMART_DSU.value):
         self.acc_type = cp_acc.vl["ACC_CONTROL"]["ACC_TYPE"]
       ret.stockFcw = bool(cp_acc.vl["PCS_HUD"]["FCW"])
@@ -208,7 +210,8 @@ class CarState(CarStateBase):
         ("PCS_HUD", 1),
       ]

# again, as above, because we have moved the 'PRE_COLLISION' message to the camera CAN bus, we need to let op know of the move
-    if CP.carFingerprint not in (TSS2_CAR - RADAR_ACC_CAR) and not CP.enableDsu and not CP.flags & ToyotaFlags.DISABLE_RADAR.value:
+    if CP.carFingerprint not in (TSS2_CAR - RADAR_ACC_CAR) and not CP.enableDsu and \
+       not (CP.flags & ToyotaFlags.DISABLE_RADAR.value or CP.flags & ToyotaFlags.DSU_BYPASS.value):
       messages += [
         ("PRE_COLLISION", 33),
       ]
@@ -224,7 +227,7 @@ class CarState(CarStateBase):
         ("LKAS_HUD", 1),
       ]

# again, as above, but this is for the timing checks, not used in new versions of op anymore
-    if CP.carFingerprint in (TSS2_CAR - RADAR_ACC_CAR):
+    if CP.flags & ToyotaFlags.DSU_BYPASS.value or CP.carFingerprint in (TSS2_CAR - RADAR_ACC_CAR):
       messages += [
         ("PRE_COLLISION", 33),
         ("ACC_CONTROL", 33),
```

```diff
# this part is for car/toyota/interface.py
# This needs to be improved, but essentially, we are just letting openpilot know that our car has a bypass installed
# you can just add ret.flags |= ToyotaFlags.DSU_BYPASS.value to here if you want and bypass all logic because the logic doesn't work without modifying panda

diff --git a/selfdrive/car/toyota/interface.py b/selfdrive/car/toyota/interface.py
index 7fd5c9435..55532719f 100644
--- a/selfdrive/car/toyota/interface.py
+++ b/selfdrive/car/toyota/interface.py
@@ -116,6 +116,10 @@ class CarInterface(CarInterfaceBase):
     if 0x2FF in fingerprint[0] or (0x2AA in fingerprint[0] and candidate in NO_DSU_CAR):
       ret.flags |= ToyotaFlags.SMART_DSU.value


# ideally, if 0x343 is detected on the camera bus and the camera bus only, the DSU bypassed flag should be set
# but realistically, because during fingerprint, bus2 and bus0 are in parallel, this can't happen
# so AGAIN, DO NOT USE THIS BIT, since the DSU bypass is more-or-less a permanent solution anyway you probably dont need this regardless

# just add
+    ret.flags |= ToyotaFlags.DSU_BYPASS.value
# should be fine, permanently setting this flag on your branch of op.

# ------------ please disregard these --------------
#+    # 0x343 should not be present on bus 2 on cars other than TSS2_CAR unless we are re-routing DSU
#+    if (0x343 in fingerprint[2] or 0x4CB in fingerprint[2]) and candidate not in TSS2_CAR:
#+      ret.flags |= ToyotaFlags.DSU_BYPASS.value
#+
#     # No radar dbc for cars without DSU which are not TSS 2.0
#     # TODO: make an adas dbc file for dsu-less models
#     ret.radarUnavailable = DBC[candidate]['radar'] is None or candidate in (NO_DSU_CAR - TSS2_CAR)
#@@ -146,7 +150,8 @@ class CarInterface(CarInterfaceBase):
#     #  - TSS2 radar ACC cars w/ smartDSU installed
#     #  - TSS2 radar ACC cars w/o smartDSU installed (disables radar)
#     #  - TSS-P DSU-less cars w/ CAN filter installed (no radar parser yet)
# ---------- end please disregard these --------------



# here, we are letting openpilot know that, if DSU is bypassed, then it can safely send 0x343 without causing a control collision
-    ret.openpilotLongitudinalControl = use_sdsu or ret.enableDsu or candidate in (TSS2_CAR - RADAR_ACC_CAR) or bool(ret.flags & ToyotaFlags.DISABLE_RADAR.value)
+    ret.openpilotLongitudinalControl = use_sdsu or ret.enableDsu or candidate in (TSS2_CAR - RADAR_ACC_CAR) or \
+                                       bool(ret.flags & ToyotaFlags.DISABLE_RADAR.value) or bool(ret.flags & ToyotaFlags.DSU_BYPASS.value)
     ret.autoResumeSng = ret.openpilotLongitudinalControl and candidate in NO_STOP_TIMER_CAR
     ret.enableGasInterceptor = 0x201 in fingerprint[0] and ret.openpilotLongitudinalControl

```

```diff
# this modifies car/toyota/values.py
# we are just adding an additional flag here so this is pretty straightforward

diff --git a/selfdrive/car/toyota/values.py b/selfdrive/car/toyota/values.py
index 9989b9225..de0a40ed1 100644
--- a/selfdrive/car/toyota/values.py
+++ b/selfdrive/car/toyota/values.py
@@ -46,6 +46,7 @@ class ToyotaFlags(IntFlag):
   HYBRID = 1
   SMART_DSU = 2
   DISABLE_RADAR = 4
+  DSU_BYPASS = 512 ###########<<<<<<<<<<<<<<<<<<<<<<< add a flag here, change the number
 
   # Static flags
   TSS2 = 8
```

## Are panda Code Modifications Required? Will This Get My Device Banned?
At this stage, panda code modifications are not required as panda filters `0x343` by default on all Toyota vehicles, and your device is unlikely to get banned.

## What Fork Supports This?
At this stage, only my fork supports it. However, support for this harness is extremely easy to implement and can be done within seconds. Send this repository to your fork's maintainer(s), and they will know what to do!
