# Hardware Tools

From the bench to industrial control: the microcontrollers, the builds they go into, and the standards that govern the result.

Author

Benedict Thekkel

This site runs from the bench upward. It starts with the microcontrollers, moves through the builds they go into, and ends at the PLC, SCADA, and machine-safety layer where the same ideas are applied at plant scale.

The pages are written as decision guides rather than datasheets. Where there is a choice to make, such as which dev board or which cellular module, the page states the trade-off and gives an answer.

The electrical toolbox, switchboard components and motors are on [Electrical Docs](https://bthek1.github.io/Electrical_Docs/). For the software counterpart see [Software Tools](https://bthek1.github.io/Software_Tools/), and for what runs on the machines see [IaaS](https://bthek1.github.io/IaaS_docs/).

------------------------------------------------------------------------

## Embedded Systems

| Page | Covers |
|----|----|
| [Microcontroller Comparison](Embedded_Systems/microcontrollers.llms.md) | Pico W against XIAO ESP32-C3 against Arduino Nano 33 BLE against STM32F. Specs, firmware stacks, and a decision guide |
| [Raspberry Pi 5](Embedded_Systems/raspberry_pi_5.llms.md) | Setup, VNC, and driving the GPIO pins |
| [Pico](Embedded_Systems/0_pico.llms.md) | The board variants and the pinout |
| [Pico with C/C++](Embedded_Systems/1_pico_c.llms.md) | The full SDK workflow on Ubuntu: toolchain install, project and CMake structure, a blink example, building, flashing UF2, VS Code setup, and serial testing |
| [ESC](Embedded_Systems/0_esc.llms.md) | The translator between a throttle signal and the switching that actually spins a brushless motor |
| [ESC Firmware](Embedded_Systems/1_esc_firmware.llms.md) | The firmware side of the same component |
| [Cellular IoT](Embedded_Systems/cellular_iot.llms.md) | Choosing a module for Australia specifically. 2G and 3G are both switched off, so only 4G, 5G, LTE-M, and NB-IoT remain, Band 28 support is non-negotiable, and the 2G support these boards advertise is worthless here. Three LilyGO modules compared, with a recommendation |
| [IotaWatt](Embedded_Systems/iotawatt.llms.md) | The circuit-level energy monitor, and writing its readings into SQL |

------------------------------------------------------------------------

## Builds

| Page | Covers |
|----|----|
| [Computer Build](computer_build.llms.md) | Every part in turn: CPU, GPU, motherboard, RAM, storage, PSU, cooling, case, OS, networking, peripherals |
| [Drone Build](drone_build.llms.md) | Frame, motors, ESCs, flight controller, props, LiPo, radio link, FPV camera and VTX, GPS, and power distribution, ending in three options per category for a five-inch value build |
| [RGB Lighting](rgb_lighting_build.llms.md) | Smart RGB lighting and the power supply behind it |
| [Home Assistant](home_assistant.llms.md) | The open-source home automation platform, run locally for privacy rather than through a vendor cloud |
| [Camera](camera.llms.md) | Notes on camera hardware |

------------------------------------------------------------------------

## Industrial Automation

Where the rest of this site scales up. Same electrons, much higher consequence of getting it wrong.

| Page | Covers |
|----|----|
| [PLC](Industrial_Automation/plc.llms.md) | The scan cycle as the core mental model, then hardware architecture, the electrical reality of I/O, memory and addressing, the five IEC 61131-3 languages, ladder logic, program organisation, communications, and HMI/SCADA |
| [SCADA](Industrial_Automation/scada.llms.md) | The supervisory layer above the PLCs, which does not close control loops. The tag model, polling against report-by-exception, protocols, HMI design, historians, redundancy, security, how it differs from DCS and MES, and what to know before building one |
| [AS 4024](Industrial_Automation/as_4024.llms.md) | The Australian machine-safety standards series: its legal status, a map of the 26 parts, the 1501/1502 against 1503 trap, how ISO 13849-1 performance levels actually work, the safety-distance calculation you will actually do, and where control engineers get caught |

------------------------------------------------------------------------

## Not Covered Yet

- **No PCB design.** Nothing on schematic capture, layout, or getting a board fabricated, which is the obvious next step after the breadboard pages.
- **Power electronics is thin**: no page on supplies, converters, or battery management as subjects in their own right.
- **[Camera](camera.llms.md) is a stub**, three cells against a subject the deep learning repos depend on.
- Sensors get no coverage at all, despite being half of any embedded project.

------------------------------------------------------------------------

Back to top
