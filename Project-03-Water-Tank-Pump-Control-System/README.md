# Project 03 — Water Tank Pump Control System

## Project Overview

This project upgrades a basic water-tank level-control application into a structured industrial PLC pump-control system using an Allen-Bradley MicroLogix 1100.

The system incorporates Auto/Manual operating modes, centralized operating permissives, level-based automatic pump requests, manual pump control, delayed pump startup, process-level indication, dry-run monitoring, master-fault handling, and operator alarm indication.

Rather than allowing the level switches to directly control the pump output, the design separates process demand, operating permission, mode-specific requests, the common pump request, startup timing, and the physical pump output.

The project was developed and functionally tested in a PLC training/simulation environment using RSLogix Micro Starter Lite.

---

## Project Files

- 📄 [Full Engineering Report](./Project_03_Water_Tank_Pump_Control_System.pdf)
- ⚙️ [RSLogix 500 PLC Program](./PV3.RSS)

> The `.RSS` file contains the native RSLogix 500 ladder-logic project. RSLogix 500 or compatible Rockwell Automation software is required to open the PLC program.

---

## Key Features

- Automatic and Manual operating modes
- Auto/Manual mode-conflict detection
- Centralized Start Permissive
- System Ready status
- Level-based automatic pump demand
- Latched automatic pump request
- Non-latched manual pump operation
- Common Pump Run Request architecture
- 3-second pump startup delay
- Low-level indication
- High-level indication
- 10-second dry-run monitoring
- Latched dry-run diagnostic fault
- Master-fault handling
- Conditional master-fault reset
- Visual fault indication
- Audible alarm indication
- Controlled pump shutdown

---

## Control Architecture

The level switches do not directly control the physical pump output.

The primary control path is:

`Mode / Protective Inputs → Fault Logic → Start Permissive → System Ready → Auto/Manual Pump Request → Pump Run Request → Startup Delay → Pump Motor`

Automatic process control follows:

`Low/High Level Conditions → Auto Run Request → Pump Run Request → Startup Delay → Pump Motor`

Dry-run diagnostics follow:

`Pump Motor + Low-Level Condition → Dry Run Timer → Dry Run Fault → Master Fault → Pump Shutdown`

This architecture separates process demand from equipment command and provides defined troubleshooting points throughout the control sequence.

---

## PLC Platform

| Item | Implementation |
|---|---|
| PLC | Allen-Bradley MicroLogix 1100 |
| Programming Software | RSLogix Micro Starter Lite |
| Programming Language | Ladder Logic |
| Pump Startup Delay | 3 seconds |
| Dry-Run Detection Timeout | 10 seconds |
| Startup Timer | T4:0 |
| Dry-Run Timer | T4:1 |

---

## Physical Inputs

| Address | Tag | Function |
|---|---|---|
| I:1/0 | Start_PB | Manual-mode Start command |
| I:1/1 | Stop_PB | Automatic-request stop condition |
| I:1/2 | Reset_PB | Master-fault reset |
| I:1/3 | Auto_Mode | Automatic mode selector |
| I:1/4 | Manual_Mode | Manual mode selector |
| I:1/5 | EStop_OK | E-stop circuit healthy |
| I:1/6 | Overload_OK | Motor overload healthy |
| I:1/7 | Low_Level_Switch | Low-level process status |
| I:1/8 | High_Level_Switch | High-level process status |

## Physical Outputs

| Address | Tag | Function |
|---|---|---|
| O:2/0 | Pump_Motor | Pump motor command |
| O:2/1 | Running_Lamp | Commanded-running indication |
| O:2/2 | Fault_Lamp | Master-fault indication |
| O:2/3 | Alarm_Horn | Audible master-fault indication |
| O:2/4 | Low_Level_Lamp | Low-level indication |
| O:2/5 | High_Level_Lamp | High-level indication |

---

## Automatic Operation

In Auto mode:

1. The PLC verifies the protective, fault, and mode conditions.
2. `Start_Permissive` becomes true when the implemented operating conditions are healthy.
3. Auto mode and Start Permissive establish `System_Ready`.
4. Low Level true with High Level false allows `Auto_Run_Request` to latch.
5. The automatic request establishes the common `Pump_Run_Request`.
6. `Delay_Timer T4:0` begins timing.
7. After 3 seconds, `Pump_Motor` energizes.
8. `Running_Lamp` follows the commanded pump output.
9. When High Level becomes true, the automatic request is removed and the pump stops.

The automatic request is also removed by the implemented Stop condition, an active master fault, or removal of Auto mode.

---

## Manual Operation

Manual mode uses a separate non-latched request.

`Manual Mode + Start PB + System Ready → Manual Run Request`

The operator command must therefore remain present for the Manual request to remain active.

Auto and Manual requests are subsequently combined into the common `Pump_Run_Request`, allowing both operating modes to share the startup timer and physical pump-output logic.

---

## Level Monitoring

The system provides direct process-state indication:

`Low_Level_Switch → Low_Level_Lamp`

`High_Level_Switch → High_Level_Lamp`

These indications allow the level-switch states to be observed independently from the pump command.

---

## Dry-Run Monitoring

Dry-run monitoring is implemented using `T4:1`.

While:

`Pump_Motor = ON`

and:

`Low_Level_Switch = FALSE`

the 10-second Dry Run Timer accumulates.

If this condition persists through the timeout:

`T4:1/DN → Dry_Run_Fault → Fault_Active`

The master fault then removes operating permission, causing the pump-control sequence to shut down.

---

## Fault Handling

The master `Fault_Active` state can be triggered by:

- Auto/Manual mode conflict
- Unhealthy E-stop status
- Unhealthy overload status
- Dry Run Fault

When `Fault_Active` is latched:

- Start Permissive is removed
- System Ready drops
- Pump Run Request drops
- Pump Motor turns OFF
- Fault Lamp energizes
- Alarm Horn energizes

Master-fault reset is permitted only when the implemented reset conditions are satisfied.

### Implementation Note

The final ladder contains a latched `Dry_Run_Fault B3:1/0`, but the final RSLogix implementation does not show an OTU instruction for that bit.

This portfolio documentation intentionally preserves the implemented PLC logic rather than claiming a Dry Run Fault reset function that is not present in the final program.

---

## Testing & Validation

The final PLC program was functionally tested for:

- System Ready operation
- Auto/Manual mode conflict
- Master-fault reset
- Automatic low-level pump request
- 3-second startup delay
- Pump startup
- High-level automatic shutdown
- Stop behavior
- Auto-mode removal
- Manual pump operation
- Low-level indication
- High-level indication
- E-stop monitoring
- Overload monitoring
- Dry-run timer operation
- Dry-run fault response
- Visual and audible master-fault indication

**17 defined functional tests passed.**

Testing represents functional PLC program validation in the project development/simulation environment rather than commissioning of a physical pumping system or validation of a safety-rated machine-control system.

---

## Engineering Lessons

This project demonstrates several important controls-engineering principles:

- Converting process conditions into internal equipment requests rather than directly driving outputs
- Separating Auto and Manual operating behavior
- Using a maintained internal request for automatic process control
- Using a non-latched request for operator-controlled Manual operation
- Centralizing operating permissives
- Sharing downstream equipment-control logic between operating modes
- Separating process indication from equipment commands
- Using timer-based diagnostics to identify abnormal process behavior
- Consolidating abnormal conditions into a master-fault architecture
- Tracing faults from upstream operating conditions through the complete pump-command path


---

## Version 1 → Version 2 Development

The original Version 1 project established the foundational water-tank level and pump-control logic. Version 2 develops that foundation into a more structured industrial pump-control system.

- 📄 [View Original Version 1 Project](./Version-1/Project_03_Version_1.pdf)

Version 2 expands the original application with Auto/Manual operating modes, centralized operating permissives, System Ready logic, separate process-demand and pump-command layers, delayed pump startup, level indication, dry-run monitoring, master-fault handling, and controlled shutdown and recovery.

---

## Industrial Relevance

Automatic tank and pump control is widely used in:

- Water and wastewater systems
- Chemical processing
- Food and beverage production
- Utility systems
- Process manufacturing
- Storage and transfer systems

The project demonstrates level-based automatic demand, Manual operation, operating permissives, delayed equipment startup, process indication, dry-run monitoring, fault handling, and controlled shutdown.

This is a PLC training and portfolio implementation. A production pumping system would normally require additional engineering such as motor protection, pump feedback, pressure or flow instrumentation where appropriate, field-device diagnostics, applicable safety-rated systems, and physical commissioning.
