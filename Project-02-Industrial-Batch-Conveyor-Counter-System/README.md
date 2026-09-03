# Project 02 — Industrial Batch Conveyor Counter System

## Project Overview

This project upgrades a basic conveyor batch-counting application into a structured industrial PLC-controlled conveyor system using an Allen-Bradley MicroLogix 1100.

The system integrates Auto/Manual operation, operating permissives, delayed conveyor startup, configurable production quantities, photoeye-based product counting, automatic batch completion, jam detection, master-fault handling, and controlled recovery.

The project was developed and functionally tested in a PLC training/simulation environment using RSLogix Micro Starter Lite 8.30.

---

## Project Files

- 📄 [Full Engineering Report](./Project_02_Industrial_Batch_Conveyor_Counter_System.pdf)
- ⚙️ [RSLogix 500 PLC Program](./PROJECT2.RSS)

> The `.RSS` file contains the native RSLogix 500 ladder-logic project. RSLogix 500 or compatible Rockwell Automation software is required to open the PLC program.

---

## Key Features

- Automatic and Manual operating modes
- Auto/Manual mode-conflict detection
- Centralized Start Permissive
- System Ready status
- Latched automatic run request
- Non-latched manual operation
- Common conveyor run architecture
- 3-second conveyor startup delay
- Configurable batch quantity
- Photoeye-based product counting
- One-shot product detection
- Automatic batch completion
- Automatic conveyor stop at target quantity
- Batch Complete indication
- Controlled batch reset
- 10-second no-product jam detection
- Latched jam fault
- Master-fault handling
- Audible fault indication
- Controlled fault recovery

---

## Control Architecture

The conveyor motor is not controlled directly from the operator Start command.

The primary control path is:

`Mode / Protective Inputs → Fault Logic → Start Permissive → System Ready → Auto/Manual Request → Conveyor Run → Startup Delay → Conveyor Motor`

Production control then follows:

`Conveyor Motor → Photoeye → One-Shot → Batch Counter → Batch Complete → Conveyor Stop`

Jam diagnostics follow:

`Conveyor Motor → Photoeye Inactivity → Jam Timer → Jam Fault → Master Fault → Conveyor Shutdown`

This architecture separates operator requests, equipment commands, production states, and diagnostic functions.

---

## PLC Platform

| Item | Implementation |
|---|---|
| PLC | Allen-Bradley MicroLogix 1100 |
| Programming Software | RSLogix Micro Starter Lite 8.30 |
| Programming Language | Ladder Logic |
| Startup Delay | 3 seconds |
| Jam Detection Timeout | 10 seconds |
| Product Counter | C5:0 |
| Configurable Batch Target | N7:0 |

---

## Physical Inputs

| Address | Tag | Function |
|---|---|---|
| I:1/0 | Start_PB | Operator start command |
| I:1/1 | Stop_OK | Auto-operation stop circuit |
| I:1/2 | Reset_PB | Fault/batch reset |
| I:1/3 | Auto_Mode | Automatic mode selector |
| I:1/4 | Manual_Mode | Manual mode selector |
| I:1/5 | EStop_OK | E-stop circuit healthy |
| I:1/6 | Overload_OK | Overload circuit healthy |
| I:1/7 | Photoeye | Product detection |

## Physical Outputs

| Address | Tag | Function |
|---|---|---|
| O:2/0 | Conveyor_Motor | Conveyor motor command |
| O:2/1 | Running_Lamp | Conveyor-running indication |
| O:2/3 | Alarm_Horn | Master-fault indication |
| O:2/4 | Batch_Complete_Lamp | Batch-completion indication |

---

## Operating Sequence

### Automatic Mode

1. The PLC verifies E-stop, overload, fault, and mode conditions.
2. `Start_Permissive` becomes true when the implemented operating conditions are healthy.
3. Selecting Auto mode establishes `System_Ready`.
4. Pressing Start latches `Auto_Run_Request`.
5. The request establishes the common `Conveyor_Run` signal.
6. `Startup_Delay T4:0` begins timing.
7. After 3 seconds, `Conveyor_Motor` energizes.
8. Products passing the Photoeye are counted.
9. When the configured target is reached, `Batch_Complete` latches.
10. The conveyor stops automatically.
11. The Batch Complete Lamp indicates the completed production batch.

### Manual Mode

Manual mode uses a non-latched `Manual_Run_Request`.

The request remains active only while the required Manual operating conditions and Start command remain present. Auto and Manual modes share the downstream `Conveyor_Run`, startup-delay, motor, counting, and diagnostic logic.

---

## Configurable Batch Control

The required production quantity is stored in:

`N7:0 — Target Quantity`

An unconditional MOV instruction transfers:

`N7:0 → C5:0.PRE`

This separates the production setpoint from the counter instruction itself and provides a natural future interface for HMI-based batch quantity entry.

Products are counted through:

`Conveyor_Motor → Photoeye → ONS B3:2/0 → CTU C5:0`

The motor qualification prevents product counts while the conveyor is stopped, while the one-shot ensures that a sustained Photoeye signal generates only one count.

---

## Batch Complete Control

When:

`C5:0.ACC = C5:0.PRE`

the counter Done condition causes `Batch_Complete B3:0/6` to latch.

Batch Complete:

- Stops the conveyor
- Removes/prevents the automatic run request
- Prevents Manual operation
- Energizes the Batch Complete Lamp
- Remains active until deliberate reset

The reset sequence clears the counter before unlatching the Batch Complete state.

---

## Jam Detection

Jam/process-flow monitoring is implemented with `Jam_Timer T4:1`.

While the conveyor is running and the Photoeye remains inactive:

`Conveyor_Motor + No Photoeye → T4:1`

If no product is detected for 10 seconds:

`T4:1/DN → Jam_Fault → Fault_Active`

The master fault then removes operating permission and activates the alarm response.

---

## Fault Handling

The master `Fault_Active` state can be triggered by:

- Auto/Manual mode conflict
- Unhealthy E-stop status
- Unhealthy overload status
- Jam fault

Fault reset is accepted only after the implemented healthy conditions have been restored.

`Alarm_Horn O:2/3` provides the common audible master-fault indication.

---

## Testing & Validation

The final PLC program was functionally tested for:

- Ready-state logic
- Auto/Manual mode conflict
- Automatic request latching
- 3-second startup delay
- Conveyor operation
- Photoeye product counting
- One-shot count protection
- Configurable batch quantity
- Automatic batch completion
- Conveyor shutdown at target
- Batch Complete indication
- Batch reset
- Jam timer operation
- Jam fault detection
- Jam recovery
- E-stop monitoring
- Overload monitoring
- Auto Stop behavior
- Auto-mode removal
- Manual operation

**20 defined functional tests passed.**

Testing represents functional PLC program validation in the project development/simulation environment rather than physical machine commissioning or safety-system validation.

---

## Engineering Lessons

This project demonstrates several important controls-engineering concepts:

- Separating operator requests from physical outputs
- Combining Auto and Manual requests into common downstream control
- Centralizing permissive and readiness logic
- Using one-shots for event-based product counting
- Separating production setpoints from ladder configuration
- Treating Batch Complete as an operating state rather than only an indication
- Integrating production states into equipment control
- Using timers to detect abnormal process behavior
- Designing latched diagnostic faults and deliberate recovery
- Troubleshooting the complete process path rather than only the counter or motor output

---

## Industrial Relevance

The concepts demonstrated in this project are applicable to:

- Manufacturing conveyors
- Packaging systems
- Warehouse automation
- Food processing
- Pharmaceutical production
- Parcel handling
- Assembly operations
- Material-handling systems

Configurable production targets are particularly relevant because industrial batch quantities are commonly supplied by an HMI, recipe system, production order, or higher-level control system.

This project is a training and portfolio implementation. A production conveyor system would require application-specific safety-rated stopping functions, guarding, motor protection, sensor validation, equipment coordination, and physical commissioning.
