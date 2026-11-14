# Chapter 3 — Reimplementing DaVinci: From Keil RTX to STM32 Arduino

*DrinkStation Engineering Journey · Source-based engineering case study*

## 3.1 The modernization problem

The original DaVinci beverage controller was already functional when I inherited it. Its STM32 firmware, developed in the Keil environment, controlled a real commercial machine. The problem was not that the equipment had stopped working: the development and service model needed to evolve to support network connectivity, remote monitoring, remote firmware maintenance, and easier ongoing extension.

In 2025 I completed my STM32 Arduino-based modernization engineering work. This repository records that work; subsequent STM32CubeIDE development is outside the scope of my contribution. The Arduino implementation was a modernization prototype, **not** a claim that Arduino became the final product toolchain.

## 3.2 What changed—and what did not

| Concern | Historical Keil / STM32 snapshot | 2025 STM32 Arduino implementation |
|---|---|---|
| MCU family | STM32F10x | STM32F103RC (`platformio.ini`) |
| Development model | Keil and RTX APIs (`RTL.h`) | PlatformIO, Arduino framework |
| Application scheduling | RTX startup task and four worker tasks | Arduino `setup()` and repeatedly executed `loop()` |
| Machine control | Task-oriented dispensing, thermal, valve, and I/O logic | Main coordination plus dedicated component classes |
| Local communications | PC/USB communication functions in historical source | `Serial2` and `SerialUSB` command inputs; JSON responses |
| Persistent settings | EEPROM-related logic in historical source | Explicit `EEPROM.get` / `EEPROM.put` configuration handling |
| IoT integration | Not designed around the later network-service architecture | Serial interface intended for an external IoT communication module |

This table compares observable source structures. It does not establish exact functional equivalence, identical timing, or a verified one-to-one port of every historical behavior.

## 3.3 Reorganizing the controller around machine functions

The modern repository has a `src/main.cpp` coordinator and dedicated modules:

```text
src/
├── main.cpp                 # Setup, loop, operating-state coordination, commands
├── CompressorControl.cpp/.h # Compressor behavior
├── HotTankControl.cpp/.h    # Hot-water thermal control
├── AgitatorPumpControl.cpp/.h
├── FilterStatusCheck.cpp/.h
├── temp_read.cpp/.h         # Temperature acquisition
├── button.cpp / Button.h
├── led.cpp / Led.h
├── DateTime.cpp/.h
└── AppTypes.h
```

This separation matters in a physical machine: thermal control, dispensing, buttons, status indicators, and communication must coexist without treating the dispenser as a generic IoT sensor.

## 3.4 Startup sequence and persistent machine state

`setup()` in `src/main.cpp` initializes pins, LEDs, the RTC, EEPROM address offsets, serial communication, the flow meter, current temperature configuration, and machine checks. The code reads persistent settings such as JSON-mode selection, total dispensed volume since filter reset, and the last filter-reset time. It also includes a conditional factory-reset path when the stored filter-reset time is nonpositive.

This is more than a demonstration sketch: the startup path has to reconstruct the machine's operating context before normal dispensing control begins.

## 3.5 The repeated control loop

The `loop()` function visibly coordinates:

- Temperature acquisition and general machine processing (`readTemperature()`, `run()`).
- Hot-water preset and child-safety checks.
- Chiller-bucket management and agitator-pump control.
- Serial command intake, safety-mode handling, and flow-rate updates.
- Filter-status tracking.
- Conditional hot-tank and compressor control.
- User-interface LED behavior associated with operating modes.

For example, the main loop invokes the specialized controllers conditionally:

```cpp
if (hotTankControl.isHotTankEnabled) {
  hotTankControl.run(ntc, currentTempConfig);
}

if (systemMode != chillerBucketRefill && isCompressorControlEnabled) {
  compressorControl.run(ntc, currentTempConfig);
}
```

These calls demonstrate explicit integration of thermal subsystems into the controller's operating state. They do **not** alone prove deterministic loop timing, scheduler equivalence with RTX, or formal safety certification.

## 3.6 Serial commands as the bridge to connected services

`readSerial()` accepts newline-delimited commands from both `Serial2` and `SerialUSB` and forwards them to `handleSerialCommand()`. The command handler recognizes operations such as `iot`, `temp`, temperature-setting commands, `flowmeter`, and automatic dispensing commands.

Responses are serialized with ArduinoJson, for example in temperature-status and dispensing-statistics handlers:

```cpp
json["hotTankMax"] = currentTempConfig.HotTankMax;
json["hotTankMin"] = currentTempConfig.HotTankMin;
serializeJson(json, Serial2);
Serial2.print('\n');
```

`platformio.ini` sets the serial monitor to **115200 baud** and includes ArduinoJson as a dependency. The repository README describes the intended architecture as **STM32 machine controller ↔ serial/JSON ↔ external IoT module**.

**Important distinction:** the observed command parser is primarily text-command based; JSON is visibly used for structured responses. It would be misleading to describe every inbound message as a JSON request without further evidence.

## 3.7 Why separate the machine controller from networking?

The modernization preserves the STM32's responsibility for physical control and exposes an interface that a separate ESP32 communication module can use for higher-level services. This makes a useful engineering boundary:

```text
Network / management services
             |
       ESP32 module
             |
      Serial commands
      JSON responses
             |
       STM32F103RC
             |
  Valves / pumps / thermal control / sensors / UI
```

This is an **architectural interpretation supported by the repositories**, not proof that every possible network operation or remote update scenario was exercised in a deployed machine. AquaMesh's bootloader design is examined separately in Chapter 4.

## 3.8 What this case study demonstrates

The central achievement was not merely replacing Keil syntax with Arduino syntax. It was modernizing a functioning STM32-controlled commercial product while retaining its physical-machine responsibilities and introducing a clearer interface for connected-device services.

The source demonstrates:

1. Understanding and extending an inherited embedded controller.
2. Reorganizing machine logic into recognizable functional modules.
3. Coordinating dispensing, thermal control, safety-related state, and user interaction.
4. Exposing operational data and commands through serial communications.
5. Preparing the controller to participate in a broader IoT service architecture.

The author reports completing and testing the modernization on the actual equipment. This chapter does not invent test measurements, failure rates, certification results, or formal test logs that are not present in the supplied archive.

## 3.9 Evidence and publication boundaries

**Modern source:** [DaVinci Arduino 2025](https://github.com/johntsaiainow/davinciarduino2025)

- `platformio.ini`: board/framework, ArduinoJson, upload/debug configuration, serial speed.
- `src/main.cpp`: `setup()` (around line 148), `loop()` (around line 184), `readSerial()` (around line 601), `handleSerialCommand()` (around line 616), JSON status handlers (around lines 800–840).
- `src/CompressorControl.cpp`, `src/HotTankControl.cpp`, `src/AgitatorPumpControl.cpp`, `src/FilterStatusCheck.cpp`: subsystem implementations.
- `README.md`: product purpose and intended IoT architecture.

**Historical comparison:** privately supplied `User.zip` / `User/main.c`, discussed in [Chapter 2](02-Legacy-STM32-Controller.md). The inherited historical source should not be republished without confirming rights.

**Next:** Chapter 4 — AquaMesh: Building the ESP32-to-STM32 Bootloader Bridge.
