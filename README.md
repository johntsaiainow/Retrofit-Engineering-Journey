# Retrofit Engineering Journey

## Modernizing a Working STM32 Beverage System for the IoT Era

**An engineering case study of legacy firmware modernization, embedded control, connectivity, firmware serviceability, and PCB design.**

The DrinkStation DaVinci was already a functioning STM32-based commercial beverage system when I took over its Keil firmware. The engineering challenge was to make a working product more maintainable and better suited to connected-device requirements—not to claim the original design or replace a broken machine.

My completed modernization work included an STM32 Arduino-based firmware prototype and related ESP32 communication, firmware-service, and PCB engineering. Subsequent STM32CubeIDE development is outside the scope of this case study.

## Chapters

1. [Engineering History and Architecture](chapters/01-Engineering-History.md)
2. [Understanding the Original STM32 Controller](chapters/02-Legacy-STM32-Controller.md)
3. [Firmware Modernization: Keil RTX to STM32 Arduino](chapters/03-Firmware-Modernization.md)
4. [AquaMesh ESP32-to-STM32 Bootloader Bridge](chapters/04-AquaMesh-Bootloader-Bridge.md)
5. [ESP32 Communication Module PCB](chapters/05-ESP32-Communication-PCB.md)
6. [Engineering Lessons Learned](chapters/06-Engineering-Lessons-Learned.md)

## Primary Engineering Repositories

- [DaVinci Arduino 2025](https://github.com/johntsaiainow/davinciarduino2025)
- [AquaMesh Basic Bootloader Bridge](https://github.com/johntsaiainow/AquaMesh-Basic-Bootloader-Bridge)
- [ESP32 Communication Module PCB](https://github.com/johntsaiainow/ESP32-Communication-Module-PCB)

## Historical Evidence and Limitations

The preserved original STM32/Keil firmware contains a header dated November 21, 2008. Its original monitoring application existed, but **the monitoring program's source code has been lost**. The archive of legacy source is used as a historical reference and is **not redistributed** here. Repository code and KiCad files support analysis of architecture and implementation, but do not by themselves prove certification, deployment volume, or every test outcome.

## Scope

This repository contains documentation only. It links to the three existing engineering repositories instead of copying their source or proprietary historical material. Descriptions distinguish documented implementation, historical first-person account, and engineering review observations.
