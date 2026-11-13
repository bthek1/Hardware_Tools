# Drone Build

Drone Build

Author

Benedict Thekkel

## **1. Quadcopter Frame**

- **Frame Size**: Choose based on your requirements (racing, freestyle, camera drone). Common sizes are 150mm (micro), 250mm (freestyle/racing), 450mm (camera/freestyle).
- **Material**: Carbon fiber is lightweight and durable. Avoid plastic unless for toy drones.
- **Mounting**: Ensure the frame supports the components you’re planning to use (motor mounting pattern, flight controller size, etc.).

------------------------------------------------------------------------

## **2. Motors**

- **Motor Size**: Defined by numbers (e.g., 2205). First two digits refer to the stator diameter; the second two are stator height.
- **KV Rating**: Determines motor speed. Higher KV = faster motor, but less torque. Choose based on propeller size and battery.
- **Type**: Brushless motors are most common due to efficiency and durability.

------------------------------------------------------------------------

## **3. Electronic Speed Controllers (ESC)**

- **Amperage Rating**: Choose ESCs that support the motor’s current draw (check motor datasheets). Add a 20-30% safety margin.
- **Number of ESCs**: 4 individual ESCs or a 4-in-1 ESC board. 4-in-1 simplifies wiring but may be harder to replace individually.
- **Firmware**: BLHeli_S or BLHeli_32 are common for advanced features and performance

------------------------------------------------------------------------

## **4. Flight Controller (FC)**

- **Processor**: F7 or F4 processors are common. F7 provides more future-proofing and features.
- **Gyros**: Look for FCs with high-performance gyros for smoother flight. MPU6000 is reliable for most builds.
- **UART Ports**: Ensure enough ports for your accessories like GPS, telemetry, and receiver.
- **Firmware**: Betaflight or iNav for FPV racing/freestyle, or ArduPilot for more advanced features.

------------------------------------------------------------------------

## **5. Propellers**

- **Size**: Propeller size should match the motor and frame size. Common sizes are 5-inch for freestyle and racing, 6-8 inch for medium, and larger for camera drones.
- **Pitch**: A higher pitch increases speed but reduces flight time. Choose based on your flying style.
- **Material**: Nylon with glass fiber is durable and affordable.

------------------------------------------------------------------------

## **6. Battery (LiPo)**

- **Cell Count**: Defined as 3S, 4S, etc. More cells mean higher voltage (3S = 11.1V, 4S = 14.8V).
- **Capacity (mAh)**: Larger capacity gives longer flight time but increases weight. Balance capacity with weight for your drone size.
- **C Rating**: Defines discharge rate. Higher C rating = more current output. Choose at least 30C-50C for racing drones.

------------------------------------------------------------------------

## **7. Radio Transmitter & Receiver**

- **Channels**: At least 6-8 channels are recommended for basic drone control.
- **Range**: Choose a transmitter with sufficient range (consider 2.4GHz or long-range 915MHz systems for FPV).
- **Receiver Type**: FrSky, FlySky, Spektrum, or Crossfire depending on your preferred brand and features.

------------------------------------------------------------------------

## **8. FPV Camera & Video Transmitter (VTX)**

- **FPV Camera**: Look for low-latency cameras with high-resolution (600TVL to 1200TVL) for a clear FPV feed.
- **Video Transmitter (VTX)**: Choose based on the required range (25mW for short-range, 200-600mW for longer ranges).
- **VTX Band**: Make sure your VTX operates on legal frequencies in your country (typically 5.8GHz).

------------------------------------------------------------------------

## **9. GPS Module (optional)**

- **Purpose**: Adds features like Return-to-Home (RTH) and position hold. Essential for camera drones and advanced features.
- **Compatibility**: Ensure it’s compatible with your flight controller firmware.

------------------------------------------------------------------------

## **10. Power Distribution Board (PDB)**

- **Current Rating**: Ensure it supports the combined current draw of all your components.
- **Built-in Features**: Some PDBs come with built-in voltage regulators or OSD (On-Screen Display) for telemetry.

------------------------------------------------------------------------

## **11. Miscellaneous Parts**

- **LEDs**: For night flying or visibility.
- **Buzzer**: Helps locate the drone in case of a crash.
- **XT60 Connectors**: Common connectors for LiPo batteries.

------------------------------------------------------------------------

Here’s the expanded version with three options per category, ordered roughly **value pick → budget pick → premium/alternative**, still biased toward efficiency and flight time:

## 5″ Value Build — Three Options Per Category

| Category | ① Value Pick | ② Budget Pick | ③ Premium / Alternative |
|----|----|----|----|
| **Frame** | TBS Source One V5 (\$25–40) — open-source, huge TPU ecosystem, cheap spares | AxisFlying Manta 5 SE (\$25–35) — includes all TPU parts, no 3D printer needed; props in view | FlyFishRC Volador VD5 (\$45–60) — deadcat-ish layout keeps props out of frame, very clean builds; heavier |
| **FC + ESC stack** | Skystars F7 Mini HDPro + KM55A 55A AM32 (\$85–100) — F722 FC, 32-bit ESC running AM32 out of the box; open-source with active development | SpeedyBee F405 V3 50A BLS (\$50–60) — flash the ESC to Bluejay (free, ~10 min via esc-configurator.com) for fully open firmware with RPM filtering; plenty of headroom for a low-KV cruiser | Holybro Tekko32 F4 60A AM32 + Kakute F405/F722 FC (\$110–130) — AM32 stock, onboard current sensing, separate boards so one failure doesn’t kill the stack |
| **Motors (6S, low-KV for efficiency)** | EMAX ECO II 2207 1700KV (\$55–65/set) — proven efficiency king at the price | T-Motor Velox V2207 1750KV (\$55–60/set) — N52 magnets, titanium shaft, ~\$15/motor | iFlight XING2 2207 1855KV (\$80–90/set) — top-tier smoothness and durability at mid-tier money; slightly thirstier KV |
| **Props** | HQ 5.1×2.5×3 (\$3–5) — low pitch, lowest cruise amps of the tri-blades | DAL Cyclone 5045C (\$2–4) — most durable, fine for practice days | Gemfan 51466 bi-blade Hurricane variants (\$3–5) — bi-blades squeeze out a bit more efficiency at the cost of grip/handling |
| **Battery — LiPo/LiHV** | GNB 6S 1500mAh LiHV (\$30–45) — energy-dense, tested well | Dogcom 6S 1300mAh (\$25–35) — strong cells, slightly heavier | Tattu R-Line 6S (\$45–60) — racer-grade, lowest sag; overkill for cruising |
| **Battery — endurance** | GEPRC/Flywoo 6S 18650 Li-ion pack (\$40–60) — ~30% more Wh/g than LiPo, ~30A limit | DIY 6S1P VTC6 pack (~\$30 in cells) — cheapest per Wh if you can spot-weld | 6S2P 21700 pack (\$70–90) — max capacity, but check it fits your frame’s battery bay (2×21700 ≈ 42mm wide) |
| **Radio** | RadioMaster Boxer ELRS (\$90–120) — full-size hall gimbals, compact body | RadioMaster Pocket ELRS (\$55–70) — hall gimbals at entry price, great second/travel radio | RadioMaster TX16S MkII ELRS (\$170–200) — flagship; touchscreen, room to grow into wings/long-range |
| **Receiver (ELRS)** | RadioMaster RP3 V2 (\$15–20) — best all-round price/performance | HappyModel EP1/EP2 (\$12–15) — tiny, cheap, fine for park-range flying | RadioMaster RP4TD / BetaFPV SuperD (\$25–35) — true diversity, better link at range — worth it if you’ll push distance |
| **FPV camera (analog)** | Caddx Ratel 2 (\$25–30) — great low light for the money | RunCam Phoenix 2 (\$25–30) — your original pick, still perfectly good | Foxeer T-Rex Mini (\$35–40) — best analog image/latency, high dynamic range |
| **VTX (analog)** | Rush Tank Solo 1.6W (\$30–35) — strong output ceiling, locked-pin safety | SpeedyBee TX800 (\$20–25) — 800mW for cheap, runs warm at full power | TBS Unify Pro32 HV (\$45–55) — the premium standby; you’re paying for pit mode polish and reliability |
| **GPS** | HGLRC M100 Mini (\$15–20) — M10 chip, fast fix, zero config | Flywoo Goku GM10 Nano V3 (\$16–20) — smallest M10 option, similar performance | Matek M10Q-5883 (\$25–30) — adds a compass, useful if you ever run iNav/position hold |
| **Buzzer** | ViFly Finder Mini (\$14–16) — 3g, ~7hr self-powered | JHE42B (\$8–10) — louder than stock, no self-power backup | ViFly Finder V2 (\$18–20) — 30hr battery + light sensor; best for actual lost-quad scenarios |

**Sample totals:** all column-② picks ≈ **\$310–350**; all column-① picks ≈ **\$380–440**; column-③ throughout ≈ **\$550–650** (all USD, including radio).

A couple of pairing notes for the flight-time goal:

- **Receiver is the one place I’d consider spending up.** If you’re cruising out to the edge of analog video range, the diversity RX (RP4TD/SuperD) is \$15 well spent — a failsafe at distance costs you the quad, and GPS rescue only helps if the link recovers.
- **Match battery to motor KV.** The 1700–1750KV options in columns ① and ② are what make the Li-ion endurance packs viable; if you go XING2 1855KV, stick to LiHV/LiPo since cruise amps creep closer to the Li-ion discharge ceiling.
- **Camera/VTX caveat:** these are all analog to keep cost down, consistent with your original list. If you ever want HD, that’s a separate ~\$130–230 decision (DJI O4/O4 Pro air unit) that replaces both rows and adds weight — which works against flight time, for what it’s worth.

------------------------------------------------------------------------

Here’s the corrected row — V4 removed (discontinued), Rush Blade removed (its ESCs run BLHeli_32, which is closed-source and abandoned), all three now open-source firmware and in production:

| Category | ① Value Pick | ② Budget Pick | ③ Premium / Alternative |
|----------|--------------|---------------|-------------------------|

Back to top
