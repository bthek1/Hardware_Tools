# ESC Firmware

ESC Firmware

Author

Benedict Thekkel

**What ESC firmware actually does**

The firmware is the software running on the ESC’s microcontroller. It determines how throttle commands from the flight controller get converted into three-phase motor drive signals — affecting throttle linearity, timing, motor noise, telemetry, and efficiency. Choosing the wrong firmware (or running outdated firmware) can cause desyncs, motor stutters, jitter, or thermal shutdown.

------------------------------------------------------------------------

**The three main options**

**BLHeli_32** was the undisputed king for years. It was the gold standard — closed-source but extremely reliable — until the developer discontinued it in 2024, meaning no new features or bug fixes will ever be released. Existing BLHeli_32 ESCs still work perfectly and will continue to do so indefinitely, but you can’t flash it to new hardware anymore.

**Bluejay** is the go-to upgrade for anyone with BLHeli_S hardware (the older 8-bit EFM8-based ESCs). It’s BLHeli_S based firmware capable of bidirectional DShot, so a great choice if you want to run RPM filtering, and it also includes a startup sound editor. Bluejay is favoured for its superior performance, robust feature set, and consistent updates over stock BLHeli_S. The one ceiling is hardware — the 8-bit EFM8 MCU struggles with very high RPM motors (50k+) or 8K PID loops with simultaneous telemetry.

**AM32** is where everything is heading. It’s an open-source firmware for brushless ESCs, born from a desire to break free from the limitations of closed-source alternatives like BLHeli_32. Its name comes from its support for ARM-based MCUs and its design target for 32-bit processors. It implements advanced Field-Oriented Control (FOC) techniques, providing exceptionally smooth operation especially at low throttle — crucial for cinematic drones — along with high RPM stability and reduced motor heating. AM32 also supports bidirectional DShot, allowing the ESC to send real-time telemetry (RPM, temperature, current) back to the flight controller over the same signal wire — critical for RPM filtering.

------------------------------------------------------------------------

**Protocols you need to know**

DShot300/600 has become the FPV mainstream, while DShot1200/2400 on high-end ESCs can provide faster and cleaner throttle signals. Bidirectional DShot is the key feature — it enables **RPM filtering** in Betaflight, where each ESC sends its motor’s exact RPM to the flight controller, which calculates the fundamental motor frequency and its harmonics, then applies narrow notch filters that track those frequencies in real time, removing only motor noise while filter latency drops dramatically. Note that bidirectional DShot requires DShot300 or DShot600 — DShot150 does not leave enough time in the frame for telemetry data to return.

AM32 also supports **Extended DShot Telemetry (EDT)**, which interleaves temperature, voltage, and current data into the eRPM telemetry frames, enabling ESC health monitoring through the same signal wire without requiring a separate serial telemetry connection.

------------------------------------------------------------------------

**The bottom line in 2026**

If you’re buying new ESCs, target AM32 or Bluejay — both are actively maintained and support RPM filtering. BLHeli_32 ESCs still work great, but the firmware won’t receive updates.

If you already have BLHeli_32 ESCs, there’s no immediate need to switch. Don’t rush into flashing AM32 because there’s no real benefit right now, and once you flash AM32 there’s no going back — it’s better to wait and make an informed decision.

Back to top
