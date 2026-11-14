# DrinkStation Engineering Journey

## Modernizing a Working STM32 Beverage System for the IoT Era

**Engineering case study | Legacy firmware modernization, embedded control, connectivity, and hardware integration**

> A working machine is not necessarily a future-ready machine. The challenge was to extend a functioning embedded system into the connected-device era without losing sight of its original purpose: reliable physical control.

## 1. Background: A Working System with Growing Limitations

The DrinkStation DaVinci beverage dispensing system already had functional STM32-based firmware developed in the Keil environment. I took over that existing firmware rather than designing the original controller from scratch. The legacy source archive includes a `config.h` header bearing a November 21, 2008 date, and the source includes Keil RTX-style task declarations.

The problem was not that the original firmware could not operate the machine. It was that the software and service model needed to evolve. Network connectivity, remote monitoring, remote firmware updates, and long-term maintainability had become important requirements for the IoT era.

In 2025, I undertook and completed my portion of the DaVinci modernization engineering work. The STM32 Arduino implementation was a prototyping approach within that work; STM32CubeIDE was identified as a subsequent development direction, outside the scope of the work documented here.

This case study focuses on **work I performed and artifacts that are available**, rather than attributing later development to me.

## 2. Engineering Objectives

The modernization effort addressed four related needs:

1. **Connectivity:** provide a practical interface between the equipment controller and network-connected services.
2. **Observability:** make machine status and diagnostic information accessible beyond the local control panel.
3. **Serviceability:** create a route for firmware maintenance without requiring direct access to the STM32 programming interface for every update.
4. **Maintainability:** organize the firmware and communication hardware so that new capabilities could be developed without collapsing all responsibilities into the original controller.

These are the motivations and design objectives. They should not be mistaken for claims that every proposed cloud service, deployment configuration, or security certification was completed.

## 3. System Architecture: Separate Physical Control from Connectivity

```text
                   Network / Wi-Fi
                         |
                         v
              +---------------------+
              | ESP32 communication |
              | and service layer   |
              | HTTP / MQTT / WS    |
              +----------+----------+
                         |
             UART / JSON | BOOT0 / NRST
                         |
                         v
              +---------------------+
              | STM32 DaVinci       |
              | machine controller  |
              +----------+----------+
                         |
           +-------------+-------------+
           |             |             |
       Dispensing     Thermal       Interface
       pumps/valves   systems       and safety
       CO2/flow       hot tank      buttons/LEDs
                       compressor    status
```

The core architectural decision was to keep **machine control on STM32** while giving **ESP32 responsibility for connectivity and firmware-service functions**. This division is documented in the DaVinci and AquaMesh project READMEs. The diagram represents the intended system relationship; it is not a claim that every repository snapshot represents a single production build.

## 4. Engineering Artifacts

### 4.1 Original STM32 / Keil Firmware — Historical Baseline

The preserved legacy source (`User.zip`) contains `main.c`, `config.h`, GPIO-related definitions, queue code, and STM32 interrupt files. `main.c` includes `RTL.h` and declares multiple `__task` functions, consistent with a Keil RTX-style multitasking design.

**Why this matters:** modernization began with understanding an operating, existing control system. This was not a greenfield microcontroller exercise.

The original source archive is treated as a historical reference. Its presence alone does not establish the date of every revision or authorize public redistribution of proprietary source.

### 4.2 [DaVinci Arduino 2025](https://github.com/johntsaiainow/davinciarduino2025) — Controller Modernization

The repository identifies the **DrinkStation DaVinci 20 Plus** controller and targets **STM32F103RC** with the Arduino framework and PlatformIO. Its `platformio.ini` configures ST-Link for programming/debugging and a 115200-baud serial monitor.

The project describes and organizes functions including:

- Beverage dispensing, pumps, valves, CO₂, and flow measurement
- Compressor, hot-tank, agitator, and temperature control
- Machine operating modes, configuration, and safety handling
- Buttons, LEDs, filter status, and machine status reporting
- Serial communication with JSON-formatted information

Source files include `CompressorControl.cpp`, `HotTankControl.cpp`, `AgitatorPumpControl.cpp`, `FilterStatusCheck.cpp`, and `temp_read.cpp`, alongside the central `main.cpp`.

**Engineering significance:** the prototype provides a maintainable way to revisit a complex physical controller while preparing an interface to external IoT functions.

### 4.3 [AquaMesh Basic Bootloader Bridge](https://github.com/johntsaiainow/AquaMesh-Basic-Bootloader-Bridge) — Remote Firmware Service

The ESP32 bridge connects network-facing firmware management to an STM32 system UART bootloader. The repository describes:

- HTTP firmware upload and LittleFS storage
- WebSocket progress reporting and MQTT status/control
- ESP32 firmware updates through ArduinoOTA
- STM32 bootloader entry through `BOOT0` and `NRST`
- UART firmware transfer, followed by return to application boot

The documented ESP32 connections are GPIO 23 (`BOOT0`), GPIO 22 (`NRST`), GPIO 16 (UART RX), and GPIO 17 (UART TX). The STM32 UART settings are **115200, 8E1**.

**Engineering significance:** a separate service controller can manage firmware updates without forcing network protocol handling into the main machine-control firmware.

**Security boundary:** network-accessible firmware updating requires authentication, firmware integrity/authenticity checks, recovery planning, and operational safeguards. The existence of a bridge is not, by itself, evidence that a production-grade secure OTA system has been certified or deployed.

### 4.4 [ESP32 Communication Module PCB](https://github.com/johntsaiainow/ESP32-Communication-Module-PCB) — Hardware Integration

The KiCad project contains `DS-IOT-Module.kicad_sch`, `DS-IOT-Module.kicad_pcb`, and supporting component libraries and 3D models.

Its README identifies an **ESP32-DEVKITC-32U**-based communication board with:

- UART to an external MCU
- STM32 `BOOT0` / `NRST` control
- I²C expansion and OLED support
- Local buttons and Wi-Fi manager reset
- JST connectors for external interfaces

**Engineering significance:** the service architecture was represented not only in firmware but also in a dedicated communication-module hardware design.

The presence of KiCad design files establishes the design artifact; manufacturing qualification, board revisions, and measured electrical results should be documented separately if published.

## 5. What This Project Demonstrates

This work illustrates several practical engineering capabilities:

- **Legacy system comprehension:** starting from a functioning commercial embedded product rather than assuming a clean-sheet redesign.
- **Hardware–firmware co-design:** coordinating controller behavior, serial protocols, boot-control signals, and a physical PCB interface.
- **System-level partitioning:** separating time-sensitive machine control from network and service functions.
- **Service-oriented thinking:** designing for diagnosis, maintainability, and firmware lifecycle needs.
- **Engineering continuity:** using the earlier STM32 implementation as a foundation for a modernized connected-device architecture.

The modernization engineering work described here was completed as my contribution. This case study does not claim ownership of the original 2008 design, subsequent STM32CubeIDE development, or undocumented production certifications.

## 6. Repositories and Evidence

| Artifact | Primary evidence |
|---|---|
| Original STM32 / Keil source archive | `User/main.c`, `User/config.h`, STM32 interrupt and queue files (historical reference; not linked publicly) |
| [DaVinci Arduino 2025](https://github.com/johntsaiainow/davinciarduino2025) | `README.md`, `platformio.ini`, `src/` |
| [AquaMesh Basic Bootloader Bridge](https://github.com/johntsaiainow/AquaMesh-Basic-Bootloader-Bridge) | `README.MD`, `platformio.ini`, `src/main.cpp` |
| [ESP32 Communication Module PCB](https://github.com/johntsaiainow/ESP32-Communication-Module-PCB) | `README.md`, `DS-IOT-Module.kicad_sch`, `DS-IOT-Module.kicad_pcb` |

## 7. Chapter 2 — Understanding the Original Controller

The detailed source-grounded analysis is available in [Chapter 2 — Understanding the Original DaVinci Controller](02-Legacy-STM32-Controller.md). It examines the Keil RTX task architecture, startup, watchdog, communications, control responsibilities, and the boundaries of what the surviving source proves.

## 8. Chapter 3 — Firmware Modernization

[Chapter 3 — Reimplementing DaVinci: From Keil RTX to STM32 Arduino](03-Firmware-Modernization.md) compares the historical controller with the 2025 STM32 Arduino implementation, including module boundaries, setup/loop coordination, EEPROM state, and serial command/JSON reporting.

## 9. Further Chapters

The following sections can be expanded as engineering evidence and personal project history are organized:

- **Chapter 4 — Building the Bootloader Bridge:** firmware-transfer protocol, failure handling, and recovery behavior
- **Chapter 5 — Designing the ESP32 PCB:** documented in the linked hardware case study
- **Chapter 6 — Integration and Validation:** actual bench and machine test procedures, findings, and lessons learned

---

*Documentation note: This article distinguishes the author's historical account from details observable in the provided source archives. It does not present unprovided test logs, certifications, performance measurements, or subsequent team work as verified facts.*

## Chapter 4 — AquaMesh Bootloader Bridge

The AquaMesh bridge uses an ESP32 as a network-connected service gateway for STM32 firmware updates. The repository implements GPIO-driven bootloader entry, UART protocol transactions, firmware staging in LittleFS, and HTTP, WebSocket, MQTT, and ArduinoOTA interfaces. The case study also documents source-visible reliability and security limitations rather than claiming unverified production guarantees.

[Read Chapter 4 — AquaMesh Bootloader Bridge](04-AquaMesh-Bootloader-Bridge.md)

### Historical note: the original monitoring program

The original DaVinci system included monitoring software, but its source code has been lost. The preserved STM32 firmware and the modern repositories can be studied directly; the missing monitoring program cannot be reconstructed or evaluated at source level from the available materials.

## Chapter 5 — ESP32 Communication Module PCB

The KiCad schematic and board layout document the physical ESP32 service interface for the DrinkStation modernization: external UART, STM32 boot/reset access, I²C expansion, local controls, and connectors. This chapter separates verified design artifacts from unprovided certification or measurement results.

[Read Chapter 5 — ESP32 Communication Module PCB](05-ESP32-Communication-PCB.md)

## Chapter 6 — Engineering Lessons Learned

[Read Chapter 6 — Engineering Lessons Learned](06-Engineering-Lessons-Learned.md).
