# STM32 FreeRTOS Motor Monitor

A FreeRTOS-based health monitoring and protection ECU for a fan/motor, running on an STM32F407. It samples vibration and supply current, detects faults with debouncing, switches the load to a safe state via a relay, and reports telemetry and diagnostic trouble codes (DTCs) over CAN. Fault handling is verified with an automated hardware-in-the-loop (HIL) test script running on a Linux CAN node.

**Hardware:** STM32F407G-DISC1 (on-board LIS3DSH accelerometer), INA219 current sensor, 5 V fan, relay module, SN65HVD230 CAN transceiver, Raspberry Pi / USB-CAN adapter as tester node.

🚧 **Work in progress.** Requirements and verification plan: [docs/requirements.md](docs/requirements.md)
