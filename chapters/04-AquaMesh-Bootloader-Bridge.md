# Chapter 4 — AquaMesh: Building an ESP32-to-STM32 Bootloader Bridge

*DrinkStation Engineering Journey — Embedded Connectivity and Firmware Serviceability*

## 1. Why a Firmware Bridge?

The original DaVinci controller was a working STM32-based commercial equipment controller. The modernization project aimed to retain STM32 responsibility for physical machine control while introducing connectivity and more practical remote service capabilities.

AquaMesh Basic Bootloader Bridge documents one part of that work: an ESP32-based network gateway that can place an STM32 into its built-in UART system bootloader and transfer an application firmware image. The project README identifies AquaMesh / PRO2 devices as its target. This chapter describes the preserved repository snapshot; it does not imply that every function in the snapshot was deployed together in a single production configuration.

An important historical qualification: the original DaVinci system **did have a monitoring program**, but its source code has been lost. The modern bridge should not be described as the first monitoring capability ever provided by the equipment.

## 2. Architectural Separation

```text
   Browser / management client / MQTT broker
                     |
           Wi-Fi / TCP-IP network
                     |
               +-----------+
               |   ESP32   |
               | HTTP      |
               | WebSocket |
               | MQTT      |
               | ArduinoOTA|
               | LittleFS  |
               +-----+-----+
                     |
             UART2 + BOOT0 + NRST
                     |
               +-----v-----+
               |   STM32   |
               | System    |
               | bootloader|
               +-----------+
                     |
             Application firmware
             / equipment control
```

The design separates network-facing maintenance functions from the STM32 equipment controller. This is a design boundary, not a claim that the current code implements authenticated, fail-safe remote updates.

## 3. Hardware Interface

The repository targets an **ESP32 DOIT DevKit V1**. `src/main.cpp` defines the following interface:

| Function | ESP32 pin |
| --- | --- |
| STM32 BOOT0 | GPIO 23 |
| STM32 NRST | GPIO 22 |
| UART RX | GPIO 16 |
| UART TX | GPIO 17 |

The UART is initialized with `uart.begin(115200, SERIAL_8E1, STM32_UART_RX, STM32_UART_TX)`: 115200 baud, eight data bits, even parity, one stop bit.

The `enterBootloader()` function raises BOOT0, pulses NRST low, then releases reset and clears pending UART bytes. That sequence requests the STM32's ROM bootloader rather than the normal application.

**Hardware integration considerations:** the repository alone does not establish the board-level reset circuit, BOOT0 biasing, voltage compatibility, isolation, or behavior when the ESP32 itself resets. These are relevant review points for a production hardware design.

## 4. Firmware Update Sequence

The implementation in `src/main.cpp` follows this sequence:

1. Receive a binary image via the HTTP upload handler and store it as `/firmware.bin` in LittleFS, or open a previously stored image in response to an MQTT command.
2. Drive BOOT0 high and reset the STM32 into its system bootloader.
3. Send the UART synchronization byte `0x7F` and wait for ACK `0x79`.
4. Send erase command `0x43` with its complement, then request mass erase.
5. Starting at `0x08000000`, send write-memory command `0x31`, the destination address and checksum, then firmware data in buffers of up to 256 bytes.
6. Publish progress through WebSocket and MQTT as bytes are written.
7. On the success path, lower BOOT0 and pulse NRST to reboot the STM32 into application firmware.

The source contains separate helpers `waitAck()`, `sendCommand()`, `sendAddress()`, `eraseFlash()`, and `writeBlock()`. This decomposition makes the bootloader transaction easier to trace than one monolithic flashing routine.

## 5. Network and Maintenance Interfaces

| Interface | Implementation in repository |
| --- | --- |
| HTTP | `/`, `/upload`, `/log`, `/download-log`, `/clear-log` |
| WebSocket | Port 81; progress and error broadcast via `wsSend()` |
| MQTT | Command subscription `pro2/flash/cmd`; status/progress/error publishing |
| ArduinoOTA | ESP32 firmware update support; hostname `pro2-0000` |
| mDNS | `pro2-0000.local` discovery |
| LittleFS | `/firmware.bin` image staging and `/flash.log` logging |

The PlatformIO configuration uses the Arduino framework, `esp32doit-devkit-v1`, LittleFS, and ESP OTA upload via `pro2-0000.local`.

**Two different update paths matter:** ArduinoOTA updates the ESP32 bridge; the STM32 UART bootloader path updates the downstream STM32. They should not be conflated.

## 6. Engineering Review: What the Snapshot Demonstrates and What It Does Not

The preserved code demonstrates the firmware-bridge design and implementation of core protocol operations. It does **not**, by itself, prove every failure scenario or security property was validated. Specific source-visible review findings include:

- **Failure recovery:** `flashFirmware()` returns early on synchronization, erase, or write failure without executing the BOOT0-low and STM32-reset sequence used on success. A production version should have a common cleanup path and explicit recovery policy.
- **Result propagation:** `flashFirmware()` returns `void`. The HTTP upload handler sends `"Flashing complete."` after calling it even if the routine reported an error through WebSocket/MQTT. A structured success/failure result would make API responses accurate.
- **Firmware validation:** the visible flow does not check a cryptographic signature, image authenticity, target identity, or post-flash integrity before reboot. Network reachability must not be mistaken for secure update authorization.
- **Final write block:** `writeBlock()` accepts the final `len` as read from the file. The repository does not visibly pad the last block to a four-byte multiple or explicitly validate target bootloader constraints; this deserves protocol-level testing.
- **Bootloader compatibility:** the implementation uses the legacy erase command `0x43` and a fixed application address `0x08000000`. STM32 variants and flash layouts require explicit compatibility checks.
- **Operational resilience:** the source does not visibly implement A/B image rollback, resume-after-power-loss, or a transactional update state machine. These should not be claimed as completed features.
- **Logging:** the implementation opens `/flash.log` and writes some success-path entries, but most intermediate progress/error reporting goes to Serial, MQTT, or WebSocket rather than a complete durable audit trail.

These observations are a **review of this repository snapshot**, not a claim about all subsequent versions, physical test results, or the final commercial system.

## 7. Why This Work Matters

Remote firmware maintenance is not merely a convenience feature. For connected physical equipment, a firmware update crosses several engineering boundaries at once: network access, firmware storage, hardware reset control, bootloader protocol handling, power-failure behavior, and recovery.

The AquaMesh repository provides concrete evidence of work across those boundaries. It also illustrates a reusable architecture: keep the dedicated microcontroller responsible for physical control while a separate communications processor provides service interfaces.

For hardware advisory work, the transferable skill is not a claim of expertise in electronic door locks. It is the ability to evaluate the interaction between **hardware control signals, embedded firmware, remote maintenance, and operational reliability**.

## 8. Source References

- [AquaMesh Basic Bootloader Bridge repository](https://github.com/johntsaiainow/AquaMesh-Basic-Bootloader-Bridge)
- `README.MD` — documented design, hardware pinout, and update flow
- `src/main.cpp` — bootloader protocol, upload handling, WebSocket/MQTT, and OTA implementation
- `platformio.ini` — build target, framework, filesystem, and dependencies

*Historical context and project ownership are based on the engineer's account. Code-level observations refer specifically to the supplied repository archive.*
