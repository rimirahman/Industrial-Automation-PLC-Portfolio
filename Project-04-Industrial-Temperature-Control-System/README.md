# Project 04 — Industrial Temperature Control System

## Project Overview

This project upgrades a basic temperature-control application into a structured industrial PLC-controlled heating system using an Allen-Bradley MicroLogix 1100.

The system separates operating permission, Auto/Manual heating demand, common heater requests, startup timing, cooling control, temperature diagnostics, alarm handling, and final physical output control.

The project was developed and functionally tested in a PLC training/simulation environment using RSLogix Micro Starter Lite.

---

## Project Files

- 📄 [Full Engineering Report](./Project_04_Industrial_Temperature_Control_System.pdf)
- ⚙️ [RSLogix 500 PLC Program](./PV4.RSS)

> The `.RSS` file contains the native RSLogix 500 ladder-logic project. RSLogix 500 or compatible Rockwell Automation software is required to open the PLC program.

---

## Key Features

- Automatic and Manual operating modes
- Auto/Manual mode-conflict detection
- Centralized Start Permissive
- System Ready status
- Latched automatic heating request
- Non-latched manual heating request
- Common Heater Run Request architecture
- 3-second heater startup delay
- Independent cooling-fan control
- High-temperature monitoring
- 5-second high-temperature alarm delay
- Out-of-range temperature detection
- Latched sensor-failure diagnostic
- Master-fault handling
- Conditional fault recovery
- Visual fault indication
- Audible alarm indication
- Final heater output-level fault and overtemperature interlocks

---

## Control Architecture

The process temperature does not directly control the physical heater output.

The primary heating path is:

`Operating Permission → Heating Demand → Heater Run Request → Startup Delay → Final Heater Output`

Automatic heating follows:

`Temperature < Heating Setpoint → Auto Heating Request → Heater Run Request → Startup Delay → Heater`

Manual heating follows:

`Manual Mode + Start → Manual Run Request → Heater Run Request → Startup Delay → Heater`

Temperature diagnostics follow:

`Process Temperature → High Temperature / Sensor Failure → Alarm & Fault Logic → Heater Interlock`

This structure separates normal process demand from diagnostic and equipment-control functions.

---

## PLC Platform

| Item | Implementation |
|---|---|
| PLC | Allen-Bradley MicroLogix 1100 |
| Programming Software | RSLogix Micro Starter Lite |
| Programming Language | Ladder Logic |
| Process Temperature | N7:0 |
| Heating Setpoint | N7:1 |
| Cooling Reference | N7:2 |
| Heater Startup Delay | 3 seconds |
| High-Temperature Alarm Delay | 5 seconds |

---

## Physical Inputs

| Address | Tag | Function |
|---|---|---|
| I:1/0 | Start_PB | Operator Start command |
| I:1/1 | Stop_PB | Automatic heating request stop |
| I:1/2 | Reset_PB | Fault/sensor-failure recovery |
| I:1/3 | Auto_Mode | Automatic mode selection |
| I:1/4 | Manual_Mode | Manual mode selection |
| I:1/5 | EStop_OK | E-stop circuit healthy |
| I:1/6 | Overload_OK | Overload circuit healthy |

## Physical Outputs

| Address | Tag | Function |
|---|---|---|
| O:2/0 | Heater_ON | Heater command |
| O:2/1 | Cooling_Fan | Cooling command |
| O:2/2 | Lamp | Heater-command indication |
| O:2/3 | Fault_Lamp | Visual fault indication |
| O:2/4 | Alarm_Horn | Audible alarm indication |

---

## Process Data

| Address | Function |
|---|---|
| N7:0 | Temperature process value |
| N7:1 | Automatic heating setpoint |
| N7:2 | Cooling comparison reference |

---

## Automatic Heating

In Auto mode:

1. The PLC evaluates the operating and fault conditions.
2. Healthy conditions establish `Start_Permissive`.
3. Auto mode and Start Permissive establish `System_Ready`.
4. When `N7:0 < N7:1`, `Auto_Heating_Request` latches.
5. The request establishes `Heater_Run_Request`.
6. `Delay_Timer T4:0` begins timing.
7. After 3 seconds, the heater can be commanded.
8. When `N7:0 >= N7:1`, the automatic heating request unlatches.

The automatic request is also removed by Stop, an active master fault, or removal of Auto mode.

---

## Manual Heating

Manual mode uses a separate non-latched request:

`Manual Mode + Start PB + System Ready → Manual Run Request`

Auto and Manual requests are combined into the common `Heater_Run_Request`, allowing both modes to share the downstream startup timing and heater-control architecture.

---

## High-Temperature Protection

A dedicated high-temperature state is generated when:

`N7:0 > 100`

This energizes:

`High_Temperature_Condition B3:0/6`

The condition performs two important functions:

- Immediately inhibits the final Heater output
- Starts the 5-second high-temperature alarm timer

If the high-temperature condition persists for 5 seconds, the delayed alarm path energizes the Alarm Horn.

---

## Sensor Failure Detection

The PLC monitors the process value for the implemented valid range.

`N7:0 < 0 OR N7:0 > 200 → Sensor_Failure`

`Sensor_Failure B3:1/0` is latched so an out-of-range temperature event remains available for diagnostics.

The sensor diagnostic can be cleared during the Reset sequence only when:

`N7:0 >= 0 AND N7:0 < 201`

This prevents an invalid process value from being cleared while the abnormal condition remains present.

---

## Cooling Fan Control

Cooling is implemented on an independent control path.

`Cooling_Fan O:2/1` requires:

- System Ready
- `N7:1 > N7:2`
- No active Sensor Failure

This keeps the cooling function separate from the normal heater-demand path while still requiring valid operating and sensor conditions.

---

## Master Fault Handling

`Fault_Active B3:0/4` can latch from:

- Auto/Manual mode conflict
- Loss of E-stop healthy status
- Loss of overload healthy status
- Sensor Failure

When the master fault is active, operating permission is removed and the Fault Lamp and Alarm Horn provide operator indication.

Fault reset requires the implemented healthy E-stop, overload, and mode conditions.

---

## Final Heater Interlock

The final heater rung performs an additional output-level check:

`T4:0/DN + Heater_Run_Request + No Fault + No High Temperature → Heater_ON`

Therefore, even if an upstream heating request exists, the physical heater output is inhibited when:

- `Fault_Active` is true, or
- `High_Temperature_Condition` is true.

This provides a final protection check immediately before the PLC heater command.

---

## Testing & Validation

The final PLC program was functionally tested for:

- Start Permissive and System Ready
- Auto/Manual mode conflict
- Automatic heating-request latching
- Automatic shutdown at setpoint
- Manual heating request
- Common Heater Run Request
- 3-second heater startup delay
- Heater output operation
- Cooling-fan operation
- High-temperature detection
- 5-second high-temperature alarm delay
- Low out-of-range sensor failure
- High out-of-range sensor failure
- Sensor-failure and master-fault recovery
- Heater inhibition during an active fault
- Heater inhibition during high temperature

**16 defined functional tests passed.**

Testing represents PLC/simulation validation of the final ladder implementation rather than physical-machine commissioning, thermal-process validation, or safety certification.

---

## Engineering Lessons

This project demonstrates several important controls-engineering concepts:

- Separating process demand from physical equipment outputs
- Creating distinct Auto and Manual operating behaviors
- Sharing downstream control logic between operating modes
- Centralizing operating permissives
- Using process-value comparisons in structured PLC architecture
- Using timers for controlled startup and persistent-condition alarming
- Detecting implausible sensor values
- Latching diagnostic events for troubleshooting
- Separating heating and cooling functions
- Rechecking critical conditions at the physical-output level
- Designing controlled fault-recovery paths


---

## Version 1 → Version 2 Development

The original Version 1 project established the foundational temperature-based heating and cooling control logic. Version 2 develops that foundation into a more structured industrial temperature-control system.

- 📄 [View Original Version 1 Project](./Version-1/Project_04_Version_1.pdf)

Version 2 expands the original application with Auto/Manual operating modes, centralized permissives, System Ready logic, separate heating-demand and equipment-command layers, controlled heater startup, independent cooling control, high-temperature monitoring, delayed alarming, sensor-failure diagnostics, master-fault handling, and final heater output interlocks.

---
## Industrial Relevance

Temperature-control systems are common in:

- Industrial ovens
- Dryers
- Process heaters
- Tanks
- HVAC-related equipment
- Manufacturing systems
- Process industries

This project demonstrates mode management, permissives, process-value comparisons, delayed equipment startup, cooling control, overtemperature monitoring, sensor diagnostics, alarms, and final output interlocks.

A production implementation could additionally require analog scaling, transmitter diagnostics, PID control, equipment feedback, independent overtemperature protection, and application-specific safety engineering.
