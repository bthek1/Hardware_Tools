# SCADA

**SCADA = Supervisory Control and Data Acquisition**: the layer above the PLCs. It doesn’t close control loops — it polls distributed controllers, stores the results, draws them on screens, raises alarms, and lets an operator issue setpoints. The controllers keep running if SCADA dies; that separation is the entire architecture.

Author

Benedict Thekkel

## Architecture

    Field devices ── sensors, valves, breakers, VFDs, meters
         │  4-20 mA / discrete / IO-Link
    PLC / RTU / IED ── local control, keeps running standalone
         │  Modbus / DNP3 / IEC 60870-5-104 / IEC 61850 / EtherNet-IP
    Front-end / Gateway / OPC UA server ── protocol translation, tag namespace
         │
    SCADA server(s) ── tag engine, alarm engine, scripting, redundancy pair
         ├── Historian (time-series DB)
         ├── HMI / operator clients (thick, thin, web, mobile)
         └── Northbound: MES, ERP, reporting, cloud

**RTU vs PLC**: an RTU is a controller optimized for remote telemetry — low power (solar/battery), wide temperature, built-in cellular/radio modem, store-and-forward buffering, timestamping at source, and event-driven reporting. A PLC is optimized for fast deterministic control. Modern devices blur the line; SCADAPack, Bristol, Kingfisher sit on the RTU side.

**IED** (Intelligent Electronic Device) is the substation term: protection relays, meters, tap changers — devices with their own protection logic that SCADA merely observes and occasionally commands.

------------------------------------------------------------------------

## The tag model

The core abstraction is a **tag**: named, typed, timestamped, quality-flagged value.

    Plant/Area2/Pump101/Status      Bool   true   Good   2026-08-11T04:22:31.442Z
    Plant/Area2/Pump101/Amps        Float  14.7   Good   ...
    Plant/Area2/Pump101/RunHours    Float  8821.3 Good   ...

Three things distinguish this from `SELECT value FROM sensors`:

1.  **Quality** is first-class. `Good / Bad_CommFailure / Uncertain_LastKnownValue / Bad_ConfigError`. A stale value that looks fine is how people get hurt. Never render a Bad-quality tag as a number without visual indication.
2.  **Timestamp source** matters. Timestamped at the RTU (accurate, needs NTP/GPS sync at every site) vs at the SCADA server on receipt (easy, wrong by the transport latency). Sequence-of-events analysis after a grid trip needs 1 ms source timestamps.
3.  **UDTs / templates.** You define `PumpType` once with its members, instantiate it 200 times, and build one screen template bound to an indirect tag path. If you’re hand-writing 200 near-identical screens you’ve already lost.

------------------------------------------------------------------------

## Polling vs report-by-exception

- **Polling**: server asks every device for every register on a cycle. Simple, predictable load, wasteful. Fine on a LAN.
- **Report-by-exception (RBE) / unsolicited reporting**: device pushes only on change beyond deadband, or on event. Essential over metered cellular links. DNP3 and IEC 60870-5-104 are built around this; Modbus is not.
- **Deadband** (analog change threshold) and **scan class** (fast/slow/on-demand groups) are the two knobs you tune to keep a link inside budget.

------------------------------------------------------------------------

## Protocols

| Protocol | Domain | Notes |
|----|----|----|
| **Modbus RTU/TCP** | universal | Registers only — no types, no timestamps, no quality, no security. Big-endian word order is *not* consistently implemented; expect to byte-swap floats. Still the default because everything speaks it. |
| **DNP3** | North American utilities, water | Real data model: static + event data, source timestamps, unsolicited reporting, class 0/1/2/3 polling, time sync. Secure Authentication v5 exists and is rarely enabled. |
| **IEC 60870-5-101/-104** | European/Asian utilities | Equivalent role to DNP3. -101 is serial, -104 is TCP. |
| **IEC 61850** | substation automation | Object-oriented model (Logical Nodes), SCL config files, MMS for client/server, **GOOSE** multicast for sub-4 ms peer-to-peer trip signals, Sampled Values for digitized CT/VT. Complex and genuinely well-designed. |
| **OPC UA** | everything modern | Platform-independent, typed information model, browsable namespace, built-in TLS + certificates + user auth, pub/sub since 1.04. This is the correct northbound interface. |
| **OPC DA/Classic** | legacy | DCOM-based. Windows-only, firewall-hostile, dying. Use a UA wrapper. |
| **MQTT + Sparkplug B** | IIoT | Push, broker-based, tiny payloads, birth/death certificates give you state awareness. Excellent over cellular and NAT. The modern telemetry answer. |
| **BACnet / KNX / M-Bus** | building automation, metering | Adjacent world, occasionally integrated. |

------------------------------------------------------------------------

## HMI design — the part everyone does badly

The 1990s aesthetic (photorealistic tanks, rainbow pipes, animated bitmaps) actively harms operators. The current standard is **high-performance HMI** (ISA-101):

- Grey scale background and equipment. Colour reserved *exclusively* for abnormal conditions.
- Analog values shown with context — trend sparkline, normal-range band — not bare digits.
- Layered navigation: Level 1 plant overview → Level 2 unit → Level 3 detail → Level 4 diagnostics.
- Consistent symbol library and colour semantics across every screen in the plant.

**Alarm management (ISA 18.2 / EEMUA 191)** is a discipline in itself. Targets: **≤ 6 alarms/hour steady state, ≤ 150 in the 10 minutes after an upset.** Real plants routinely run at 10× that, operators learn to ignore the banner, and then Texas City happens. Practices that matter: every alarm must have a defined operator response, priority distribution roughly 80/15/5 low/med/high, deadband and on-delay to kill chatter, first-out / cause-effect suppression, shelving with expiry, and rationalization documentation. Alarm floods are a design failure, not an operator failure.

------------------------------------------------------------------------

## Historian

Time-series storage optimized for one write pattern and one read pattern.

- **Swinging-door / dead-band compression** — stores a value only when a new sample would fall outside a tolerance corridor of the interpolated line. 10–100× reduction with bounded error. Understand it before you trust a reconstructed trend.
- Query semantics you need: raw, interpolated-at-interval, aggregates (min/max/avg/stddev/time-weighted-avg), and *time-weighted* is the correct default for process values.
- Products: OSIsoft/AVEVA PI (the incumbent, expensive, ubiquitous in oil & gas and utilities), Canary, InfluxDB, TimescaleDB, ClickHouse. Ignition’s built-in historian writes to Postgres/MSSQL and is perfectly adequate for most plants.
- Retention tiering: 1 s raw for 90 days → 1 min rollup for 2 years → 15 min forever, is a typical shape.

------------------------------------------------------------------------

## Redundancy and availability

- **Hot standby SCADA pair** with synchronized tag state and automatic failover; clients reconnect transparently.
- **Redundant comms paths** — primary fibre, backup cellular, with automatic failover per-device.
- **Store-and-forward** at the edge: RTU buffers timestamped data during a link outage and backfills the historian on reconnect. Without this, every comms blip is a hole in your data forever.
- Control authority always degrades *downward*: PLC keeps running its last valid setpoints when SCADA is gone. Design so an unreachable SCADA is boring.

------------------------------------------------------------------------

## Security

SCADA is the layer with the internet-facing risk, so this is not optional.

- **Purdue model / ISA-95 levels 0–5**, with an **industrial DMZ** between level 3 (site operations) and level 4 (enterprise IT). Data flows out through the DMZ; nothing initiates inbound to control.
- **IEC 62443** is the governing standard family — zones, conduits, security levels SL1–SL4.
- Legacy protocols have **no authentication whatsoever**. Anyone with L2 access can issue a breaker-open command over unsecured DNP3. Security is topological, not cryptographic, on the field side.
- Precedent: **Stuxnet** (2010, centrifuge damage via PLC firmware manipulation), **Ukraine 2015/2016** (grid outages via stolen HMI credentials, then a purpose-built IEC-104 tool), **Colonial Pipeline 2021** (IT ransomware, but OT shut down precautionarily — which is the common real-world pattern), **Oldsmar** (remote-access software on an HMI). Nearly all start with remote access and flat networks, not exotic zero-days.
- Practical baseline: unidirectional gateway or firewalled DMZ, no vendor remote access without jump host + MFA, disable unused protocol servers, signed firmware, patching windows negotiated a year ahead because you cannot reboot a running plant.

------------------------------------------------------------------------

## SCADA vs DCS vs MES

- **DCS** (Emerson DeltaV, Honeywell Experion, ABB 800xA, Siemens PCS 7): single-vendor, tightly integrated, engineered as one database from field device to screen. Suits continuous processes — refineries, chemicals, power stations. Higher cost, higher integrity, less flexibility.
- **SCADA**: multi-vendor, loosely coupled, supervisory over independent controllers. Suits discrete and distributed applications.
- **MES**: production orders, batch genealogy, OEE, quality — level 3, sits above SCADA, business-facing.

The distinction has eroded; modern SCADA + PLCs does most of what a small DCS does, at a fraction of the price.

------------------------------------------------------------------------

## Vendors

| Product | Notes |
|----|----|
| **Ignition** (Inductive Automation) | The one to learn. Java-based, unlimited-tag server licensing, Jython 2.7 scripting, web-deployed Perspective clients, native MQTT/Sparkplug, Postgres/MSSQL backend. The only one that behaves like a normal software platform. |
| **AVEVA System Platform / InTouch** (ex-Wonderware) | Huge installed base, object-oriented (ArchestrA), heavy licensing. |
| **Siemens WinCC / WinCC OA** | WinCC for TIA-integrated plant HMI; WinCC OA (ex-ETM PVSS) for very large distributed systems — CERN runs on it. |
| **Rockwell FactoryTalk View SE** | Tight AB integration, Windows-centric, painful outside the ecosystem. |
| **GE/Emerson iFIX & CIMPLICITY** | Long-lived, common in water and utilities. |
| **COPA-DATA zenon** | Strong in energy and pharma, good IEC 61850 support. |
| **Open source**: Rapid SCADA, ScadaBR/Mango, OpenPLC + Node-RED + Grafana | Fine for learning and low-consequence monitoring. Not for anything with a safety or regulatory dimension. |

------------------------------------------------------------------------

## Building one yourself — read this before you start

Your stack (Django + DRF + React + Postgres + Redis) maps deceptively well: Postgres/TimescaleDB as historian, Redis as the live tag cache, Channels/WebSockets for push to a React dashboard, `pymodbus` or `asyncua` for the driver layer, Celery for scheduled polls. You could have a working prototype in a fortnight.

What you’d be re-implementing badly, in rough order of pain: quality propagation, store-and-forward with historical backfill, alarm state machines (unacked-active / acked-active / unacked-cleared, plus shelving and suppression), redundancy and failover, tag templating with indirect binding, per-device driver quirks, sub-second update fan-out to 30 clients, and audit trails for operator commands (21 CFR Part 11 if pharma, and utilities have their own).

Reasonable split: **build it yourself for monitoring and analytics; buy Ignition the moment an operator can issue a command that moves something.** The liability line is the write path, not the read path. If you do go custom, use OPC UA (`asyncua`) as your southbound interface rather than raw Modbus — you get types, browsing, subscriptions, and TLS for free, and any vendor gateway will expose one.

Given your CV/ML interest, the genuinely interesting layer is above SCADA: historian data → feature pipeline → anomaly detection and soft sensors, fed back as advisory-only tags. That’s where the value is and where nobody’s liability is on the line.

------------------------------------------------------------------------

Back to top
