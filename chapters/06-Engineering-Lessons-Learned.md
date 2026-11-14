# Chapter 6 — Engineering Lessons Learned

## Modernizing a Working Embedded Product

*DrinkStation Engineering Journey — retrospective engineering case study*

## 1. The Starting Point Was a Working Machine

The original DrinkStation DaVinci STM32 controller, developed using Keil, was already operational when I took over its firmware. The challenge was not to rescue a nonfunctional product or claim authorship of its original design. The challenge was to modernize a working commercial equipment platform whose connectivity, monitoring, firmware maintenance, and development workflow were increasingly difficult to extend.

The surviving legacy firmware provides evidence of an STM32-based controller with Keil RTX-style tasks and machine-control responsibilities. The original monitoring application also existed, but its **source code has been lost**. Its absence from the archive is not evidence that the original system had no monitoring functionality.

## 2. Modernization Without Discarding the Control Platform

The 2025 work retained STM32 as the equipment controller and used the STM32 Arduino framework and PlatformIO for a modernization prototype. The source repository documents dispensing, thermal-control, status, safety, and serial communication modules. The work described in this series represents my completed modernization engineering contribution; it should not be confused with later STM32CubeIDE development, which was outside my responsibility.

**Lesson:** When existing hardware performs its physical task, modernizing the software architecture and service interfaces can be a more focused engineering strategy than replacing the entire controller.

## 3. Separate Physical Control From Network Services

The related projects establish a useful division of responsibility:

| Layer | Project evidence | Responsibility |
| --- | --- | --- |
| Equipment control | [DaVinci Arduino 2025](https://github.com/johntsaiainow/davinciarduino2025) | STM32 machine logic, sensors, pumps, valves, thermal controls, status |
| Firmware service | [AquaMesh Bootloader Bridge](https://github.com/johntsaiainow/AquaMesh-Basic-Bootloader-Bridge) | ESP32 network-facing firmware staging and STM32 UART bootloader access |
| Communication hardware | [ESP32 Communication Module PCB](https://github.com/johntsaiainow/ESP32-Communication-Module-PCB) | KiCad design exposing serial, boot/reset, expansion, and local service interfaces |

This is a documented architectural relationship, not a claim that every repository snapshot was installed together in a single production revision.

**Lesson:** A network-connected service processor can be developed independently of time-sensitive equipment control. The separation helps make responsibilities reviewable, but does not automatically establish safety, cybersecurity, or fault isolation.

## 4. Firmware Updates Are Hardware–Software Transactions

The AquaMesh code demonstrates why remote firmware updates are more than transferring a file. The update path coordinates network upload, local storage, ESP32 GPIO control of STM32 BOOT0/NRST, UART synchronization, erase/write commands, and application restart.

The reviewed source snapshot also reveals boundaries that matter in any engineering review: failure-path reset cleanup, truthful API result reporting, firmware authenticity, image compatibility, power interruption, and recovery. These are review findings, not claims that the completed DrinkStation modernization failed in the field.

**Lesson:** Update workflows should be evaluated end-to-end, including failure states and physical recovery, not only the successful flash sequence.

## 5. A PCB Is a Service Interface, Not Just a Circuit Drawing

The KiCad project records a concrete ESP32 communication module with external connectors, UART and bootloader-related interfaces, I²C expansion, and local controls. This makes wiring and maintenance assumptions inspectable by other engineers.

The preserved design files do not independently establish ERC/DRC results, manufacturing qualification, certification, or deployment quantities. Those claims should be tied to separate records if they are published later.

**Lesson:** Design artifacts become stronger engineering evidence when their intended interfaces and validation boundaries are explicit.

## 6. Historical Source Preservation Matters

The surviving Keil firmware, the Arduino prototype, the AquaMesh source, and the KiCad files provide different views of the same modernization effort. The lost original monitoring-program source illustrates the limits of reconstructing a legacy product from incomplete archives.

For future maintenance and technical storytelling, preserve:

- Source snapshots with revision and ownership context;
- Hardware schematics, PCB files, connector maps, and firmware configuration;
- Test procedures, logs, and the conditions under which results were observed;
- Architecture decisions, including alternatives considered and known limitations.

This list is a retrospective recommendation, not a statement that every listed record survives from the DrinkStation project.

## 7. Transferable Engineering Practice

The work demonstrates experience with **legacy firmware comprehension, embedded control, hardware/firmware interfaces, connectivity integration, firmware service architecture, and PCB design review**. These are relevant to connected commercial equipment, including products that require dependable field maintenance and third-party hardware review.

The case study does **not** claim direct experience with electronic strikes, maglocks, Wiegand/OSDP, access-control certification, or Good2Go-specific hardware. Those are separate product-domain skills.

## 8. Closing Reflection

> The original machine worked. The engineering opportunity was to help it remain useful in a connected world.

The DrinkStation modernization was not a single code rewrite. It was a systems-engineering effort spanning an existing STM32 controller, a new firmware development approach, an ESP32 maintenance bridge, and communication-module PCB design. The historical record is strongest when it respects both the work completed and the evidence that remains available.

---

**Related chapters:** [Legacy Controller](02-Legacy-STM32-Controller.md) · [Firmware Modernization](03-Firmware-Modernization.md) · [AquaMesh](04-AquaMesh-Bootloader-Bridge.md) · [ESP32 PCB](05-ESP32-Communication-PCB.md)
