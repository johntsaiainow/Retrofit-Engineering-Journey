# Chapter 2 — Understanding the Original DaVinci Controller

## 2.1 Why begin with the existing machine?

The original DaVinci controller was not a failed prototype. It was a working STM32-based beverage-control system whose original source was maintained in a Keil toolchain. I inherited this working firmware. The modernization challenge was to understand its operating behavior before introducing connectivity, remote diagnostics, and a more maintainable development path.

The preserved source archive (`User.zip`) is a **historical code snapshot**, not necessarily a complete, independently buildable Keil project: `User/main.c` includes headers such as `usb_main.h`, `hw_config.h`, `ff.h`, and `RTL.h` that are not all included in the archive. The header of `User/config.h` records **November 21, 2008** and **V1.0**. Those header entries are evidence of the documented history, not proof that every line in the snapshot dates from 2008.

## 2.2 MCU and software foundations

The source selects the STM32F10x device family by default in `User/config.h` (via `stm32f10x.h` when `STM32_CM0_SET` is not defined). It uses STM32 peripheral-library APIs such as `RCC_APB2PeriphClockCmd` and `GPIO_Init`, and the Keil RTX API via `#include <RTL.h>`.

Rather than a single loop controlling everything, the snapshot defines a startup task and four worker tasks:

```c
OS_TID os_id_init;
OS_TID os_id1, os_id2, os_id3, os_id4;

__task void task_init(void);
__task void task_A(void);
__task void task_B(void);
__task void task_C(void);
__task void task_D(void);
```

In `task_init`, the four workers are created with `os_tsk_create_user(...)`, each with an explicit stack buffer and priority argument `1`. The application subsequently starts its RTX scheduler with `os_sys_init_user(...)`. These are observable implementation details, not reconstructed pseudocode.

## 2.3 Task responsibilities visible in the source

| Task | Observed source behavior | Engineering interpretation |
|---|---|---|
| `task_init` | Creates worker tasks; performs USB initialization and FAT filesystem/file operations | Startup and service orchestration |
| `task_A` | Initializes/refreshes the independent watchdog and performs a one-time EEPROM write | Watchdog and persistence-related housekeeping |
| `task_B` | Contains the large dispensing/control state logic and manipulates valve, pump, and indicator flags | Main machine operation and control state |
| `task_C` | Calls `send_pc()` and delays | Outbound communication/reporting |
| `task_D` | Checks `Read_over2()`, calls `pctomcu2()`, refreshes watchdog, and calls `io.io_main()` | Inbound communication and physical I/O service |

These descriptions are based on the inspected function bodies. They are **not** a claim that each task has only one responsibility, nor a formal timing or safety analysis.

## 2.4 Physical-control complexity

The source shows why modernization could not be treated as merely adding Wi-Fi. `task_B` coordinates multiple modes and actuators, including cold and hot water valves and CO₂ control flags. Other code handles temperature processing and physical output behavior. The controller must continue operating a real machine while any new communications layer is added.

The code also contains a sliding-window averaging routine, `HDLB_NTC_T1(...)` / `HDLB_NTC_T2(...)`, illustrating application-side sensor processing. These routines shift samples, sum a window, and return the arithmetic mean; they are not evidence of a specific measured temperature accuracy or response time.

## 2.5 Communication and local service behavior

The historical source already includes **local communications and service mechanisms**: `send_pc()`, `Read_over2()`, `pctomcu2()`, USB initialization, and FAT filesystem operations. Therefore, describing the original firmware as having *no communications* would be inaccurate.

The modernization gap was instead the absence of the **modern network-connected service model** the project needed: remote monitoring, managed connectivity, and practical firmware servicing across connected devices. This distinction matters when telling the engineering story.

## 2.6 Watchdog and operational continuity

Both `task_A` and `task_D` include `IWDG_ReloadCounter()` calls in their loops, and `task_B` also includes watchdog servicing. This shows that watchdog operation was part of the original design. It does **not** by itself establish the fault-recovery guarantees of the deployed machine. Any future safety or timing claims should be tied to test records and hardware configuration.

## 2.7 What carried forward into modernization

The design lesson was to **preserve the responsibility of the STM32 for equipment control** while extending service capabilities through a separate ESP32 communication/bootloader layer. The later DaVinci Arduino 2025 and AquaMesh repositories show that direction, but the legacy snapshot and later repositories should not be portrayed as byte-for-byte equivalents or as a single verified production firmware image.

This is a story of *engineering continuity*: understanding working behavior, separating critical machine functions from connectivity, and making the equipment maintainable for the IoT era.

## 2.8 Source references and limits

- Historical archive: `User.zip` → `User/config.h`, `User/main.c` (particularly task declarations near lines 46–56; task implementations near lines 3353–4050; startup near line 4102).
- Modern controller: [DaVinci Arduino 2025](https://github.com/johntsaiainow/davinciarduino2025).
- Network/firmware bridge: [AquaMesh Basic Bootloader Bridge](https://github.com/johntsaiainow/AquaMesh-Basic-Bootloader-Bridge).

**Publication note:** The original source was inherited and may contain third-party or employer-owned material. This chapter discusses architecture and short illustrative snippets; do not publish the complete historical source without confirming redistribution rights.
