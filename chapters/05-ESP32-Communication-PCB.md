# Chapter 5 — ESP32 Communication Module PCB

## From the firmware architecture to physical service hardware

The DrinkStation DaVinci modernization was not limited to rewriting STM32 firmware. A separate ESP32 communication board provided a hardware path toward network connectivity, controller communications, and service access. This chapter documents the design artifacts preserved in the [ESP32 Communication Module PCB repository](https://github.com/johntsaiainow/ESP32-Communication-Module-PCB).

## 1. Engineering context

The original DaVinci controller was a working STM32-based machine controller. The modernization retained the distinction between **physical equipment control** and **network/service functions**. The ESP32 module was designed to support the latter without requiring the main controller to take on the whole networking stack.

The related projects are:

- [DaVinci Arduino 2025](https://github.com/johntsaiainow/davinciarduino2025) — STM32 controller firmware modernization prototype.
- [AquaMesh Basic Bootloader Bridge](https://github.com/johntsaiainow/AquaMesh-Basic-Bootloader-Bridge) — ESP32-to-STM32 firmware-maintenance bridge.
- [ESP32 Communication Module PCB](https://github.com/johntsaiainow/ESP32-Communication-Module-PCB) — KiCad hardware design for the communication/service interface.

These are components of one engineering modernization effort, not unrelated portfolio examples.

## 2. Hardware design files

The repository contains native KiCad project artifacts:

| Artifact | Engineering purpose |
| --- | --- |
| `DS-IOT-Module.kicad_pro` | KiCad project configuration |
| `DS-IOT-Module.kicad_sch` | Editable electrical schematic |
| `DS-IOT-Module.kicad_pcb` | Editable PCB layout |
| `PCB/` | Component libraries, footprints, and 3D models |
| `DS-IOT-Module-backups/` | Archived design backups |
| `README.md` | Project overview and interface documentation |

These files document a concrete PCB design, not merely a conceptual block diagram. Their presence alone does not establish production quantity, certification, or a successful design-rule check.

## 3. System partitioning

```text
               Wi-Fi / Network
                      |
                      v
             +------------------+
             | ESP32-DEVKITC-32U|
             | communication &  |
             | service hardware |
             +--------+---------+
                      |
         +------------+-------------+
         |            |             |
      UART TX/RX    I2C SDA/SCL   BOOT0 / NRST
         |            |             |
         v            v             v
     External MCU   OLED /       STM32 boot
                    expansion    mode & reset
```

The separation provides a useful engineering boundary: the STM32 is responsible for machine-control behavior, while the ESP32 offers network-facing services and maintenance interfaces. This is an architectural description, not a claim that all possible network services were exercised on the physical board.

## 4. Components and interfaces evidenced by the schematic

The native KiCad schematic identifies the following items:

| Schematic item | Value / description | Design relevance |
| --- | --- | --- |
| `U2` | `ESP32-DEVKITC-32U` | Main communication platform |
| `J3` | `B6B-XH-A_LF__SN_` | External connector |
| `J4` | `B4B-XH-A_LF__SN_` | External connector |
| `S3`, `S4` | Push-button components | Local hardware controls |
| Named interface | `Uart/Bootloader` | Controller communications / maintenance |
| Named interface | `I2C` | Peripheral expansion |
| Named function | `WiFi Manager Reset` | Local configuration reset provision |

The project README additionally describes an OLED interface, BOOT0/NRST signals, and 3.3 V/GND connections. Pin-level electrical compatibility, power budgets, and connector wiring should be checked against the schematic before deriving installation instructions.

## 5. Design reasoning

### Keep network responsibilities out of the primary controller

An ESP32 service module makes it possible to develop network protocols and device-management features separately from the STM32 machine-control firmware. This reduces architectural coupling, although it does not by itself guarantee isolation or fault tolerance.

### Expose firmware service signals

UART and STM32 boot/reset control signals provide the hardware interface needed by the related AquaMesh bootloader bridge. Exposing these signals is a practical serviceability decision: firmware-maintenance software cannot reliably enter a controller's system bootloader without appropriate physical control and serial connectivity.

### Use serviceable, identifiable interfaces

The repository preserves connector footprints, local switches, and the KiCad design itself. These make it possible to review connector selection, routing, and service access with a manufacturing or hardware integration partner.

## 6. Verification boundaries

This chapter is based on inspection of the repository README and native KiCad design files. It does **not** claim:

- A recorded KiCad ERC or DRC pass;
- Measured current consumption, signal integrity, or environmental reliability;
- UL/FCC certification or production manufacturing qualification;
- A specific number of manufactured or deployed boards;
- Successful testing of every firmware/network feature on this particular PCB revision.

Those statements require corresponding test reports, measurements, manufacturing records, or explicit project history. The broader DrinkStation modernization work was completed by the engineer; the scope of the available **PCB evidence** is narrower than the scope of the overall engineering effort.

## 7. Relevance to hardware technical advisory work

This project illustrates a practical hardware-review domain: translating embedded communication and firmware-service requirements into schematic connections, board layout, connectors, and accessible controls. In a technical-advisor role, the same method can be applied to evaluating third-party designs for power, wiring, maintainability, cost, and service access—without claiming prior experience in unrelated lock-specific hardware.

## 8. Historical note

The original DaVinci system had a monitoring program, but its source code has been lost. The preserved original STM32 firmware, the 2025 Arduino prototype, the AquaMesh bridge, and this KiCad board form the documented evidence for the modernization story. The missing monitoring program should not be represented as an absence of monitoring in the original product.

---

**Next:** Chapter 6 — Engineering Lessons Learned: Modernizing Working Embedded Products.
