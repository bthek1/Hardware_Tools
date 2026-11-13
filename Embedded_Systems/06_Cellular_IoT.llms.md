# Cellular IoT

**Australian network context (mid-2026):** 2G was switched off years ago (Telstra 2016, Optus 2017, Vodafone 2018) and **3G is now also gone** (Vodafone/TPG Jan 2024, Telstra + Optus Oct 2024). Only **4G/5G** remain, plus the IoT-specific **LTE-M (Cat-M1)** and **NB-IoT** networks. The band that matters most here is **Band 28 (700 MHz)**, which Telstra and Optus lean on heavily for IoT and rural/wide-area coverage. Any module you buy **must support B28** — and the `GSM/GPRS/EDGE` (2G) support these boards advertise is **useless in Australia**.

Author

Benedict Thekkel

## Quick comparison

|  | **T-A7670 R2** | **T-SIM7670G S3** | **T-SIM7070G** |
|----|----|----|----|
| Cellular module | A7670 (E / G / SA) | SIM7670G | SIM7070G |
| Network tier | **4G LTE Cat-1** | **4G LTE Cat-1** | **LTE-M / NB-IoT** |
| Throughput | 10 ↓ / 5 ↑ Mbps | 10 ↓ / 5 ↑ Mbps | ~tens–hundreds of kbps |
| Works in Australia? | ✅ **only G or SA** (not E) | ✅ global, incl. B28 | ✅ on LTE-M / NB-IoT |
| Power profile | Moderate | Moderate | **Ultra-low** (battery) |
| Camera interface | ❌ | ✅ **\[H802\] SKU only** | ❌ |
| GNSS / GPS | Optional (pick a “With GPS” SKU) | ✅ | ✅ |
| MCU | ESP32 (WROVER) | ESP32-S3 | ESP32 |
| List price | **\$32.89 USD** (≈ \$27 on AliExpress) | **\$36.02 USD** | **\$29.01 USD** |
| Best for | Cheapest general 4G data / telemetry | Camera + frequent telemetry | Battery sensor phoning home |

*Prices are vendor list (USD) as of this writing; expect roughly **\$45–80 AUD landed** after shipping/GST. Add a camera and antennas where the board doesn’t include them.*

------------------------------------------------------------------------

## T-A7670 R2

- [link](https://lilygo.cc/products/t-sim-a7670e?variant=43043706077365)

**Marketed as:** *“Wireless Module ESP32 Chip 4G LTE CAT1 MCU32 Development Board”*

A classic **ESP32 + A7670 4G LTE Cat-1** board — same data tier as the SIM7670G (**10 Mbps down / 5 Mbps up**), but the cheapest of the three and built around a regular ESP32 (WROVER) rather than an S3. It bundles the ESP32, A7670 modem, nano-SIM slot, microSD, optional GPS, an 18650 holder, solar charging and battery management on one board. Fine for periodic image uploads and steady telemetry over the network — but note **there is no onboard camera interface**, so a camera would have to be handled separately. This is the general-purpose “get data to the internet” board, not a camera platform.

### ⚠️ The variant choice is make-or-break: E / G / SA

The A7670 ships in three regional band variants. They are the *same board* — only the modem’s LTE bands differ:

| Variant | Designed for | Band 28 (AU)? | Verdict for Australia |
|----|----|----|----|
| **A7670E** | Europe / SE Asia / South America | ❌ **No** (tops out at B20) | ❌ **Do not buy** — flaky-to-dead here |
| **A7670G** | **Global** (widest coverage) | ✅ Yes | ✅ Safe choice (LilyGO’s recommended) |
| **A7670SA** | South America / **New Zealand / Australia** | ✅ Yes | ✅ Purpose-built for ANZ |

**For Brisbane: buy the G or the SA — never the E.** GPS is a separate per-SKU option (e.g. *T-A7670G R2 With GPS*), so pick a “With GPS” variant if you want GNSS.

### Hardware gotchas

- The **microSD card uses IO2 as its CS line**, which clashes with flashing — you must **remove the SD card when uploading a sketch**.
- Battery voltage can’t be read over USB (the BAT ADC only works on battery power); recommended solar input is 4.4–6 V — don’t exceed it.

**Buy (variant selector):** <https://lilygo.cc/products/t-sim-a7670e> **Docs:** <https://github.com/Xinyuan-LilyGo/LilyGO-T-A7670X>

------------------------------------------------------------------------

## T-SIM7670G S3

- [link](https://lilygo.cc/products/t-sim-7670g-s3?variant=43932694413493)

**The 4G Cat-1 board with a camera option — the best fit for image frames + frequent telemetry.**

The SIM7670G is a **4G LTE Cat-1** modem (up to **10 Mbps down / 5 Mbps up**) on the global band set, which **includes Band 28** — so it’s a safe choice on Telstra/Optus with no region-variant guesswork. Built on an **ESP32-S3**, so you also get Wi-Fi + BLE locally alongside the 4G uplink.

### Watch the SKU — two variants exist

| Feature | **\[H707\]** | **\[H802\] “Standard”** |
|----|----|----|
| MCU | ESP32-S3-WROOM-1 **N16R8** (16 MB flash / **8 MB PSRAM**) | ESP32-S3-WROOM-1 **N16R2** (16 MB flash / **2 MB PSRAM**) |
| Camera interface | ❌ | ✅ |
| QWIIC | ❌ | ✅ |
| Seamless power switching | ❌ | ✅ |
| GNSS routing to SoC + PPS | ❌ | ✅ |
| eSIM pad | ❌ | ✅ |

**If you want the camera, you must buy the \[H802\] SKU.** The catch: the camera-capable H802 has only **2 MB PSRAM** — fine for an OV2640 at VGA/SVGA JPEG, but tight for high-resolution frames (big framebuffers want 8 MB). The 8 MB-PSRAM H707 has no camera header at all.

**Buy:** <https://lilygo.cc/products/t-sim-7670g-s3>

------------------------------------------------------------------------

## T-SIM7070G

- [link](https://lilygo.cc/products/t-sim7070g?pr_prod_strat=e5_desc&pr_rec_id=722ce2adf&pr_rec_pid=7308804325557&pr_ref_pid=7372121276597&pr_seq=uniform)

**The low-power board — right country, wrong tier for camera work.**

The SIM7070G is a multi-mode **LTE-M (Cat-M1) / NB-IoT** module (with a 2G GPRS fallback that’s irrelevant in Australia). LTE-M and NB-IoT **do** run on Telstra and Optus, so it connects fine here.

The point of this board is **power, not speed**. Throughput is tiny — designed for a sensor that wakes, sends a small payload (a few readings, a status, a GPS fix), and sleeps for hours or days on a battery. A camera frame would crawl or fail to upload reliably.

- **Choose it for:** battery/solar remote sensors, small infrequent payloads, multi-year deployments where power budget dominates.
- **Don’t choose it for:** images, video, or frequent/bulky data — use a Cat-1 board instead.

**Buy:** <https://lilygo.cc/products/t-sim7070g>

------------------------------------------------------------------------

## Recommendation

**For a camera + frequent-telemetry build:** the **T-SIM7670G S3 \[H802\]** is the pick — correct Cat-1 tier, global Band 28, and the only one here with a camera interface.

**If the camera is handled separately (or you don’t need one):** the **T-A7670 R2 (G or SA, With GPS)** is the cheapest solid 4G Cat-1 board for getting telemetry to the internet — just never buy the E variant.

**If the project is really a low-power battery sensor:** the **T-SIM7070G** is the right tool; the Cat-1 boards would waste its power advantage.

> **Easiest-path alternative to all of the above:** the **Waveshare ESP32-S3-SIM7670G-4G** (same SIM7670G modem, same Cat-1 tier, same Band 28) ships as a near-complete kit — OV2640 camera, LTE + GNSS antennas, and an acrylic case all in the box — and is stocked **locally in Australia** (~\$99 AUD, same-day dispatch). Costs more than the bare LilyGO boards, but saves the offshore shipping wait and the hunt for a camera and antennas.
>
> Source: <https://iot-store.com.au/products/esp32-sim7670g-4g-lte-gnss>

------------------------------------------------------------------------

## Don’t forget (applies to any of these)

- **SIM + APN.** You need a data-capable nano-SIM and the carrier’s APN set in firmware (e.g. Telstra `telstra.internet`, ALDI `mdata.net.au`). A cheap prepaid data SIM on the Telstra or Optus network is fine to start.
- **Data costs.** Camera frames + frequent telemetry burn data fast. Keep payloads lean — MQTT over raw HTTP, compress images, batch readings, send only on change.
- **Power.** These integrated boards already handle the modem’s ~2 A current spikes (onboard power path + battery management). A bare modem wired to a plain MCU without that is the \#1 cause of random resets.
- **Software.** Quickest route is the **TinyGSM** Arduino library (gives you a `Client` for MQTT/HTTP, and supports A7670 / SIM7670 / SIM7070). The cleaner route is **ESP-IDF’s `esp_modem`** component, which brings the modem up as a PPP network interface so the standard TCP/IP stack just works.

------------------------------------------------------------------------

*Tiers at a glance: **NB-IoT / LTE-M (SIM7070G)** = sips power, tiny data → battery sensors. **4G Cat-1 (A7670 / SIM7670G)** = moderate power, real data → images + telemetry. **4G Cat-4 (SIM7600G)** = more power, fast → live video.*

------------------------------------------------------------------------

Back to top
