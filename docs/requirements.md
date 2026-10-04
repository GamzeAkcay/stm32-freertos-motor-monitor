# Requirements & Verification

Status legend: **TBD** = not yet verified. Every requirement gets a measured or tested result before it is marked done.
Verification methods: **M** = measurement (logic analyzer / timestamps), **H** = HIL test (automated, via CAN), **U** = unit test (host), **R** = review / inspection.

## 1. Timing

| ID | Requirement | Method | Result |
|---|---|---|---|
| REQ-T01 | Accelerometer is sampled with a period of 1 ms, jitter ≤ 50 µs (measured at a GPIO toggled by the sampling task) | M | TBD |
| REQ-T02 | A telemetry frame (vibration RMS, peak, fan current) is sent on CAN every 100 ms ± 1 ms | M, H | TBD |
| REQ-T03 | After a fault is confirmed (post-debounce), the relay is opened within 5 ms | M | TBD |
| REQ-T04 | After a fault is confirmed, the corresponding DTC frame appears on the bus within 10 ms | H | TBD |
| REQ-T05 | A configuration command received over CAN takes effect no later than the next processing window (≤ 100 ms) | H | TBD |

## 2. Fault handling

| ID | Requirement | Method | Result |
|---|---|---|---|
| REQ-F01 | Vibration RMS above the warning threshold for N consecutive windows raises a warning DTC; the fan keeps running | U, H | TBD |
| REQ-F02 | Fan current outside the valid range for a debounce time raises a fault DTC and enters the safe state (relay open). Thresholds defined after characterization (see §5) | U, H | TBD |
| REQ-F03 | Loss of communication with a sensor (bus error / timeout, N consecutive failures) raises a sensor DTC and enters the safe state | U, H | TBD |
| REQ-F04 | If any monitored task misses its liveness check-in, the independent watchdog resets the MCU within 500 ms; the reset cause is reported on CAN after reboot | H | TBD |
| REQ-F05 | The safe state is latched and is left only by an explicit clear command over CAN | U, H | TBD |
| REQ-F06 | Fault injection commands (test builds only) can simulate each fault above without touching the hardware | R, H | TBD |

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
| 0x100 | ECU → tester | Telemetry: vibration RMS, peak, fan current |
| 0x101 | ECU → tester | DTC event: code, status (active / latched / cleared) |
| 0x102 | ECU → tester | Boot info: reset cause |
| 0x200 | tester → ECU | Commands: set thresholds, clear faults, fault injection (test builds) |

IDs 0x7E0/0x7E8 are intentionally left free (used by the UDS bootloader project).

## 5. Open items (decided by measurement, not assumption)

- Fan current when free-running vs. blocked by hand. Many brushless fans have locked-rotor protection, so a blocked fan may draw *less* current, not more. The stall-detection rule is defined only after this measurement.
- Vibration RMS with and without an imbalance weight on a blade, with board and fan mounted on the same rigid plate.
- Whether the relay module's input triggers reliably from a 3.3 V GPIO.
