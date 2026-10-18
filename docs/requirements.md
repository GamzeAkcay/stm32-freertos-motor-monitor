# Requirements & Verification

**System:** Rollover / overtemperature safety ECU on STM32F407 + FreeRTOS. Monitors tilt and shock (LIS3DSH accelerometer) and ambient temperature (DHT22), switches the load to a safe state via a relay, and reports telemetry and diagnostic trouble codes (DTCs) over CAN.

Status legend: **TBD** = not yet verified. Every requirement gets a measured or tested result before it is marked done.
Verification methods: **M** = measurement (logic analyzer / timestamps), **H** = HIL test (automated, via CAN), **U** = unit test (host), **R** = review / inspection.

## 1. Timing

| ID | Requirement | Method | Result |
|---|---|---|---|
| REQ-T01 | The accelerometer is sampled with a period of 10 ms, jitter ≤ 50 µs (measured at a GPIO toggled by the sampling task) | M | TBD |
| REQ-T02 | A telemetry frame (tilt angle, peak acceleration, temperature) is sent on CAN every 100 ms ± 1 ms | M, H | TBD |
| REQ-T03 | After a fault is confirmed (post-debounce), the relay is opened within 5 ms | M | TBD |
| REQ-T04 | After a fault is confirmed, the corresponding DTC frame appears on the bus within 10 ms | H | TBD |
| REQ-T05 | A configuration command received over CAN takes effect within 100 ms | H | TBD |
| REQ-T06 | Reading the DHT22 does not delay any higher-priority task by more than 100 µs (no busy-waiting in the scheduler's critical path) | M | TBD |

## 2. Fault handling

| ID | Requirement | Method | Result |
|---|---|---|---|
| REQ-F01 | Tilt angle above the rollover threshold (default 45°) continuously for the debounce time (default 500 ms) raises a rollover DTC and enters the safe state | U, H | TBD |
| REQ-F02 | Acceleration magnitude above the shock threshold (default 2 g) raises a shock DTC and enters the safe state | U, H | TBD |
| REQ-F03 | Temperature above the overtemperature threshold for N consecutive readings raises an overtemperature DTC and enters the safe state | U, H | TBD |
| REQ-F04 | Loss of communication with a sensor (timeout or checksum error, N consecutive failures) raises a sensor DTC and enters the safe state | U, H | TBD |
| REQ-F05 | If any monitored task misses its liveness check-in, the independent watchdog resets the MCU within 500 ms; the reset cause is reported on CAN after reboot | H | TBD |
| REQ-F06 | The safe state is latched and is left only by an explicit clear command over CAN, and only if no fault condition is still present | U, H | TBD |
| REQ-F07 | Fault injection commands (test builds only) can simulate each fault above without touching the hardware | R, H | TBD |

## 3. Robustness

| ID | Requirement | Method | Result |
|---|---|---|---|
| REQ-R01 | Every task keeps ≥ 25 % unused stack (stack high-water mark) under worst-case test load | M | TBD |
| REQ-R02 | No heap allocation after the scheduler has started | R | TBD |
| REQ-R03 | Stack overflow and malloc failure are trapped by hooks and indicated (red LED) | R, M | TBD |
| REQ-R04 | Data shared between tasks is accessed only through queues, task notifications or a mutex | R | TBD |

## 4. CAN interface (500 kbit/s, standard IDs)

| ID | Direction | Content |
|---|---|---|
| 0x100 | ECU → tester | Telemetry: tilt angle, peak acceleration, temperature, state |
| 0x101 | ECU → tester | DTC event: code, status (active / latched / cleared) |
| 0x102 | ECU → tester | Boot info: reset cause |
| 0x200 | tester → ECU | Commands: set thresholds, clear faults, fault injection (test builds) |

IDs 0x7E0/0x7E8 are intentionally left free (used by the UDS bootloader project).

## 5. Open items (decided by measurement, not assumption)

- Noise level of the tilt angle at rest, which sets the minimum useful debounce and hysteresis.
- Peak acceleration from a deliberate tap on the table vs. normal handling, which sets the shock threshold.
- Whether the relay module's input triggers reliably from a 3.3 V GPIO.

## 6. Stretch goal (only after §1–§3 are verified)

- Linux-side gateway in C++: CAN telemetry forwarded over UDP, fault-clear and configuration commands accepted over TCP.
