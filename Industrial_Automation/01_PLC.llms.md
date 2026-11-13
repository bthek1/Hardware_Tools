# PLC

PLC = Programmable Logic Controller (assuming that, not power-line communication).

Author

Benedict Thekkel

## What it actually is

An industrial computer optimized for deterministic, always-on I/O control. The differentiator isn’t compute — it’s the execution model, the I/O electrical robustness, and 20-year serviceability. A PLC is a microcontroller with the hard parts (isolation, surge tolerance, power, watchdog, field-replaceable modules, standardized programming) already solved and certified.

------------------------------------------------------------------------

## Scan cycle — the core mental model

Everything follows from this loop:

1.  **Read inputs** → copy all physical input states into the input image table
2.  **Execute program** → top to bottom, left to right, against the *image table*, not live I/O
3.  **Write outputs** → copy output image table to physical outputs
4.  **Housekeeping** → comms, diagnostics, watchdog reset

Consequences you must internalize:

- Logic sees a frozen snapshot. An input that pulses shorter than the scan time may be missed entirely — that’s what high-speed counter inputs and hardware interrupts exist for.
- **Rung order matters.** Writing a bit on rung 50 and reading it on rung 10 introduces a one-scan delay. Reading it on rung 51 is same-scan.
- Scan time is typically 1–20 ms. It’s variable (branch-dependent) but bounded; watchdog faults the CPU if exceeded.
- No preemption within the main task by default. Long loops in ST will blow the watchdog — there’s no `while(1)`.

------------------------------------------------------------------------

## Hardware architecture

- **CPU module** — runs firmware + your program. Has run/stop keyswitch, diagnostic LEDs, retentive memory, real-time clock.
- **Power supply** — usually 24 VDC system; 120/230 VAC input.
- **Backplane/rack** — parallel bus connecting modules; or a chained connector on DIN-rail “compact” systems.
- **I/O modules** — digital in/out, analog in/out, specialty (high-speed counter, motion, thermocouple, weigh scale, serial).
- **Remote I/O** — I/O racks over fieldbus (Profinet/EtherNet/IP/EtherCAT) placed near the equipment; standard on anything nontrivial.

Form factors: modular rack (ControlLogix, S7-1500), compact/brick (S7-1200, CompactLogix, Click), PC-based soft PLC (Beckhoff TwinCAT, CODESYS runtime), and PAC (blurred PLC/PC line).

------------------------------------------------------------------------

## I/O electrical reality

- **Digital in**: 24 VDC, opto-isolated. **Sinking vs sourcing** (NPN vs PNP) — get this wrong and nothing works. Europe defaults PNP sourcing sensors / sinking inputs; Japan/Asia often NPN.
- **Digital out**:
  - *Relay* — dry contact, any voltage, slow (~10 ms), finite mechanical life, no leakage.
  - *Transistor (MOSFET)* — DC only, fast, PWM-capable, small leakage current.
  - *Triac* — AC loads, no zero-cross issues, leakage can falsely trigger sensitive inputs.
- **Analog in**: 4–20 mA (preferred — live-zero detects broken wire, immune to voltage drop), 0–10 V (short runs only), RTD (Pt100, 3/4-wire), thermocouple (needs cold-junction comp).
- **Analog out**: 4–20 mA to valve positioners, VFD speed references.
- Resolution matters: 12-bit vs 16-bit modules; raw counts scale to engineering units in software.
- **Isolation** — channel-to-channel vs group isolation. Ground loops on analog are the \#1 source of “noisy readings.”

------------------------------------------------------------------------

## Memory & addressing

| Concept                | Siemens          | Rockwell           |
|------------------------|------------------|--------------------|
| Physical input         | `%I0.0`          | `Local:1:I.Data.0` |
| Physical output        | `%Q0.0`          | `Local:2:O.Data.0` |
| Internal bit           | `%M10.0`         | tag `BOOL`         |
| Structured data        | DB (data block)  | tag / UDT          |
| Reusable code w/ state | FB + instance DB | AOI                |
| Stateless code         | FC               | Routine/JSR        |

Rockwell is fully tag-based (symbolic, no fixed addresses). Siemens supports both absolute and symbolic; use symbolic and set “optimized block access” on. **Retentive** memory survives power cycle; non-retentive clears. Deciding what’s retentive is a design decision — recipe data yes, motor run command no.

Data types: `BOOL, BYTE, WORD, DWORD, SINT, INT, DINT, LINT, REAL, LREAL, STRING, TIME, DATE`. `INT` is 16-bit in the IEC world — a common bug source for anyone coming from C. UDTs (structs) are supported and should be used heavily.

------------------------------------------------------------------------

## IEC 61131-3 languages

- **LD (Ladder)** — relay-diagram metaphor. Dominant in North America, discrete manufacturing. Electricians can troubleshoot it live at 3 AM; that’s its whole value proposition. Terrible for math and string handling.
- **FBD (Function Block Diagram)** — signal-flow blocks. Good for analog/process control, interlock chains.
- **ST (Structured Text)** — Pascal-like. Use this for anything algorithmic: state machines, recipe handling, math, arrays, communications parsing. As a software engineer you’ll want ST everywhere; resist that for interlock and motor-control logic that maintenance staff must read.
- **SFC (Sequential Function Chart)** — Grafcet-derived steps + transitions. Excellent for batch/sequential processes.
- **IL (Instruction List)** — assembly-like, deprecated in the 3rd edition. Ignore.

Best practice: ladder for device control and interlocks, ST for sequencing/math, FBD for PID loops.

------------------------------------------------------------------------

## Ladder logic essentials

- Rung = one logical expression. `-| |-` NO contact (true when bit true), `-|/|-` NC contact (true when bit false), `-( )-` coil.
- **Seal-in / latch pattern**: `Start OR Motor_Run` in series with `NOT Stop` → `Motor_Run`.
- **Wire E-stops and stop buttons as normally closed contacts**, then use an NO contact in logic. Broken wire = machine stops. This is the single most important safety convention in the field.
- **Double coil is a bug**, not a feature — the last rung wins. Each output gets exactly one write location.
- Timers: `TON` (on-delay), `TOF` (off-delay), `TP` (pulse), `RTO` (retentive, needs explicit reset). Timers count in scan-independent real time; `.DN`, `.ET/.ACC`, `.EN` members.
- Counters: `CTU`, `CTD`, `CTUD` with explicit reset.
- Edge detection: `R_TRIG`/`F_TRIG` or `ONS`/`P`/`N` contacts. Essential for “do this once” logic in a loop that runs 500×/sec.
- `SET`/`RESET` (latch/unlatch) — use sparingly; they hide state across the program and survive PLC stop/start.

------------------------------------------------------------------------

## Program organization

- **Tasks**: continuous (runs whenever there’s slack), periodic (e.g. every 10 ms — use this for PID and anything time-sensitive), event (triggered by I/O or motion). Periodic tasks preempt continuous.
- **Interrupt OBs** (Siemens: OB30-38 cyclic, OB40 hardware interrupt, OB80/82/86/121 fault handlers). If you don’t provide a fault OB, many CPUs stop on error.
- Structure code as: safety/interlocks → sequencing → device control → alarms → comms/HMI interface. Alarms and HMI mapping in dedicated routines.
- Reusable device blocks (a “Motor” FB/AOI with inputs for start/stop/interlocks and outputs for run/fault) are how you keep 50-motor machines maintainable.

------------------------------------------------------------------------

## Communications

**Fieldbus / industrial Ethernet:** - **Modbus RTU/TCP** — dead simple, no config, universally supported, no diagnostics, polled. Excellent for third-party devices. - **EtherNet/IP** (Rockwell, CIP over TCP/UDP) — implicit (cyclic I/O) + explicit (acyclic messaging). - **Profinet** (Siemens) — RT and IRT (isochronous, sub-ms jitter). - **EtherCAT** (Beckhoff) — frame-on-the-fly, microsecond determinism, best for motion. - **Profibus DP / DeviceNet / CANopen** — legacy but everywhere. - **IO-Link** — point-to-point digital link to smart sensors over standard 3-wire cable; gives you parameters and diagnostics from a proximity sensor. - **AS-i** — 2-wire power+data for simple field devices. - **OPC UA** — vendor-neutral, information-modeled, the standard upward interface to MES/IT. OPC UA PubSub + TSN is the current convergence story. - **MQTT / Sparkplug B** — report-by-exception to the cloud/broker; the “Unified Namespace” architecture. This is the modern IT-OT bridge and where your Django/Python skills plug in directly.

------------------------------------------------------------------------

## HMI / SCADA

HMI = local panel display; SCADA = supervisory, multi-machine, with historian, alarms, trending. Ignition (Java/Jython, unlimited-tag licensing, web-deployed) is the developer-friendly one. WinCC (Siemens), FactoryTalk View (Rockwell), Wonderware/AVEVA elsewhere. ISA-101 governs HMI design: grey backgrounds, colour reserved for abnormal states, no 3D gauges. Historians (PI, Canary, InfluxDB/TimescaleDB) store time-series at scale.

------------------------------------------------------------------------

## Safety — treat as a separate discipline

- **Never implement E-stop or guard-interlock functions in standard PLC logic.** Use a safety relay (simple) or a safety PLC (Siemens F-CPU, Rockwell GuardLogix, Pilz) with certified function blocks.
- Standards: **IEC 61508** (base), **ISO 13849** (PL a–e, machinery), **IEC 62061** (SIL), **IEC 60204-1** (electrical equipment of machines).
- Stop categories: **Cat 0** immediate power removal, **Cat 1** controlled stop then power removal, **Cat 2** controlled stop, power maintained.
- Dual-channel redundant wiring with cross-monitoring, and forced-guided contactor feedback (EDM).
- Safety programs are password-locked, checksummed, and require validation documentation on change.

------------------------------------------------------------------------

## Motion & process control

- **VFDs** for AC induction motors — controlled over fieldbus (speed reference + control word) or analog 4–20 mA + digital run/stop.
- **Servo drives** for positioning — PLCopen Motion Control function blocks (`MC_Power`, `MC_MoveAbsolute`, `MC_Home`, `MC_CamIn`) are standardized across vendors.
- **Encoders** — incremental (A/B/Z) into high-speed counters, or absolute (SSI/EnDat/fieldbus).
- **PID**: run in a periodic task with fixed dt. Scale to engineering units first. Practical tuning: start P-only, increase until oscillation, back off ~50%, add I to remove offset, D only for temperature/slow processes. Always implement anti-windup and bumpless auto/manual transfer — vendor PID blocks do this for you.

------------------------------------------------------------------------

## Debugging & commissioning

- **Online monitoring** — live rung highlighting is the killer feature; you see truth flow through logic in real time.
- **Cross-reference** — find every read/write of a tag. Non-negotiable on inherited code.
- **Forcing** — overrides physical I/O. Extremely dangerous, always logged, must be cleared before handover. Many sites ban it.
- **Trends/traces** — high-speed capture buffers for catching intermittent faults.
- **Diagnostic buffer** — the CPU’s own event log; first place to look on an unexpected stop.
- Simulators: PLCSIM Advanced, Studio 5000 Emulate, CODESYS built-in. Test sequences offline before touching hardware.
- FAT (factory acceptance test) → SAT (site acceptance test) → commissioning is the standard delivery flow.

------------------------------------------------------------------------

## Software engineering practice (where the industry is weak)

- Version control is poor: most projects are opaque binaries. Siemens TIA has version-control-friendly XML export (and TIA Openness API for scripting); Rockwell has L5X exports and the newer `.ACD`-adjacent tooling. Practical answer: commit exports, use vendor compare tools for diffs, keep a rigorous change log.
- No unit test culture by default. CODESYS and TwinCAT (TcUnit) support real test frameworks — use them if your platform allows.
- Change control is usually procedural, not tooled: signed change requests, backup before/after, printed as-built docs.
- Redundancy: hot-standby CPU pairs (ControlLogix Redundancy, S7-400H/S7-1500R/H) for processes that cannot stop.

------------------------------------------------------------------------

## Security (IEC 62443)

- **Purdue model** levels 0–5: field devices → control → supervisory → MES → enterprise → internet. Traffic crosses layers only through defined conduits.
- OT/IT DMZ, no direct internet on control networks, unidirectional gateways for high-security sites.
- PLCs historically have zero authentication on their control protocols. Modern CPUs add protection levels, signed firmware, and certificate-based comms — enable them.
- Stuxnet is the canonical lesson: air-gaps are not a security control, and PLC firmware integrity matters.

------------------------------------------------------------------------

## Vendors, honestly

- **Siemens** — S7-1200 (compact) / S7-1500 (modular), TIA Portal. Dominant in Europe/Asia. Deep, rigid, excellent diagnostics, heavyweight IDE.
- **Rockwell / Allen-Bradley** — CompactLogix / ControlLogix, Studio 5000. Dominant in North America. Best-in-class tag model and online editing, extremely expensive.
- **Beckhoff** — TwinCAT 3 on Visual Studio, PC-based, EtherCAT. By far the best developer experience: C++/C#/Matlab integration, real debugging, real source control. Best fit for someone from your background.
- **CODESYS ecosystem** — WAGO, Eaton, Festo, Schneider, many others. Vendor-neutral IEC 61131-3 IDE; learn it once, use it broadly.
- **Mitsubishi / Omron / Keyence** — strong in Asia, Keyence for vision-integrated.
- **Schneider (Modicon)** — process and infrastructure heavy.
- **Budget/entry**: Automation Direct Click and Productivity (cheap, free software), Arduino Opta, Unitronics.
- **OpenPLC** — open-source IEC 61131-3 runtime that runs on Raspberry Pi, ESP32, and Linux. Free way to learn ladder + Modbus without buying hardware.

------------------------------------------------------------------------

## PLC vs the alternatives

|  | Determinism | Dev experience | Field robustness | When |
|----|----|----|----|----|
| **PLC** | Hard, ms | Poor-to-fair | Excellent | Machine control, must run 10 yrs, maintainable by technicians |
| **PAC/IPC** | Hard, µs–ms | Good | Good | Motion, vision, complex algorithms |
| **DCS** | Soft, process-oriented | Fair | Excellent | Continuous process plants, thousands of analog loops |
| **RTU** | Soft | Poor | Excellent, low-power | Remote telemetry, SCADA over cellular/radio |
| **MCU (ESP32/STM32)** | Whatever you build | Excellent | You build it | Products, prototypes, cost-sensitive volume |

Coming from ESP32: a PLC is not more capable than your microcontroller — it’s less capable and vastly more reliable, because everything is constrained. The scan model removes concurrency bugs. The I/O modules remove EMC failures. The certification removes liability. That’s the trade.

------------------------------------------------------------------------

## Practical getting-started path

1.  Install **CODESYS** (free) or **OpenPLC** and run a soft PLC on a Pi. Write ladder for a start/stop/seal-in motor circuit and a traffic light SFC.
2.  Get real: an S7-1200 starter kit or an AutomationDirect Click (~AU\$200) plus a 24 V PSU and a few switches/lamps.
3.  Wire a Modbus TCP link between the PLC and a Python script — this is where your existing stack meets theirs.
4.  Add an HMI: Ignition Maker Edition is free for personal use and is a full SCADA.
5.  Read the safety standards before you ever build something that can move and hurt someone.

------------------------------------------------------------------------

## Common gotchas summary

- NC field wiring for stops; NO contact in logic.
- Double coils; last write wins.
- Rung order creating one-scan delays.
- Missing edge detection → repeated triggering every scan.
- `INT` overflow at 32,767.
- Forgetting to scale analog raw counts.
- Retentive vs non-retentive chosen by accident.
- Analog ground loops and unshielded cable run alongside VFD motor cable.
- No fault OB → CPU stops on a divide-by-zero.
- Timers preset in ms vs 100 ms base depending on platform.
- Doing safety in standard logic. Never.

------------------------------------------------------------------------

Back to top
