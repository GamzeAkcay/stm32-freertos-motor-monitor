# STM32 FreeRTOS Safety ECU

A FreeRTOS-based rollover / overtemperature safety ECU on an STM32F407. It monitors tilt and shock with the on-board accelerometer, confirms faults with debouncing, latches a safe state until it is explicitly cleared, and (planned) reports telemetry and diagnostic trouble codes (DTCs) over CAN.

Requirements are written before the code, and every requirement gets a measured or tested result: [docs/requirements.md](docs/requirements.md)

> 🚧 **Work in progress.** The status table below shows what is actually implemented and verified today.

## Status

| Feature | Requirement | Status |
|---|---|---|
| FreeRTOS setup (native API, TIM6 timebase, stack-overflow and malloc-failed hooks) | REQ-R03 | ✅ Done |
| Button ISR → queue → task, with 50 ms debounce | – | ✅ Done |
| LIS3DSH accelerometer over SPI1, periodic 10 ms sampling (`vTaskDelayUntil`) | REQ-T01 | ✅ Done (jitter not yet measured) |
| Rollover detection: > ~45° for 500 ms continuously | REQ-F01 | ✅ Done, verified by hand |
| Latched safe state, cleared only by command and only when level | REQ-F06 | ✅ Done (button used as the clear command for now) |
| Shock detection | REQ-F02 | ⏳ Planned |
| DHT22 temperature sensor, overtemperature and sensor-loss faults | REQ-F03, F04, T06 | ⏳ Planned |
| Relay output for the safe state | REQ-T03 | ⏳ Planned |
| CAN telemetry, DTCs and commands | REQ-T02, T04, T05 | ⏳ Planned |
| Independent watchdog with task liveness check-ins | REQ-F05 | ⏳ Planned |
| Automated HIL tests from a Linux CAN node | REQ-F07 | ⏳ Planned |
| Timing measurements (jitter, fault reaction time) with a logic analyzer | REQ-T01, T03 | ⏳ Planned |

## Current tasks

| Task | Priority | Trigger | Job |
|---|---|---|---|
| `ImuTask` | 4 | every 10 ms | Reads X/Y, debounces tilt, latches the rollover fault, drives the status LEDs |
| `ButtonTask` | 3 | queue (from EXTI0 ISR) | Debounces the user button and sends a "clear fault" notification to `ImuTask` |

The ISR only puts a timestamp into a queue and returns; all processing happens in tasks. Tasks talk to each other through queues and task notifications, not shared globals.

**LEDs:** green = normal, red = rollover fault latched (or accelerometer not responding at startup).

## Design notes

- **Debounce instead of instant reaction.** Shaking the board briefly pushes the X/Y reading over the threshold. A fault is confirmed only after 50 consecutive tilted samples (500 ms); a single level sample resets the counter.
- **Latched safe state.** Once confirmed, the fault stays active even if the board is put back level. It is cleared only by an explicit command, and only if the fault condition is gone.
- **First-SPI-transfer bug.** The first `WHO_AM_I` read returned `0xFF`. With CPOL = High, HAL enables the SPI peripheral on the first transfer, so SCK jumped from low to high *after* chip select was already low and the sensor lost sync. Fix: enable SPI before the first transaction.

## Lab notes

- [Lab 01 – `vTaskDelay` vs `vTaskDelayUntil`](docs/lab01-scheduling.md)

## Hardware

| Part | Role | Status |
|---|---|---|
| STM32F407G-DISC1 | MCU board, on-board LIS3DSH accelerometer, LEDs, user button | In use |
| DHT22 | Temperature and humidity | Planned |
| Relay module | Safe-state output | Planned |
| SN65HVD230 | CAN transceiver | Planned |
| Linux PC / Raspberry Pi with CAN adapter | Tester node (logging, HIL tests) | Planned |

## Repository layout

```
firmware/   STM32CubeIDE project (CubeMX-generated, user code in USER CODE blocks)
docs/       requirements, lab notes
tools/      tester-side scripts (planned)
```

## Build

Open `firmware/stm32-freertos-motor-monitor` in STM32CubeIDE, build, and flash with the on-board ST-LINK. Peripheral configuration is in the `.ioc` file (open it with STM32CubeMX).
