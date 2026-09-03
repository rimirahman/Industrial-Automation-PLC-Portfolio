# Project 01 — Industrial Motor Control System

## Project Overview

This project upgrades a basic motor start-delay application into a structured industrial PLC motor-control system using an Allen-Bradley MicroLogix 1100.

The control architecture separates operator requests, operating-mode validation, startup permissives, timing, internal commands, physical outputs, feedback monitoring, diagnostics, and equipment-use information.

The project was developed and functionally tested in a PLC training/simulation environment using RSLogix Micro Starter Lite 8.30.

---

## Project Files

- 📄 [Full Engineering Report](./Project_01_Industrial_Motor_Control_System.pdf)
- ⚙️ [RSLogix 500 PLC Program](./PROJECT1.RSS)

> The `.RSS` file contains the complete RSLogix 500 ladder-logic project. RSLogix 500 or compatible Rockwell Automation software is required to open the native PLC project file.

---

## Key Features

- Automatic and Manual operating modes
- Auto/Manual mode-conflict detection
- Centralized startup permissive
- System-ready indication
- Latched automatic run request
- Manual hold-to-run operation
- 5-second motor startup delay
- Internal run-command architecture
- Dedicated motor-contactor output
- Motor feedback verification
- 3-second feedback validation timer
- Latched master-fault handling
- Visual and audible fault indication
- Completed-start event counting
- Runtime accumulation
- Controlled fault reset and recovery

---

## Control Architecture

The motor is not controlled directly from an operator Start command.

Instead, the PLC evaluates the control path:

`Protective Inputs / Mode Selection → Fault Logic → Start Permissive → System Ready → Operating Request → Start Request → Startup Delay → Run Command → Motor Contactor → Feedback Verification`

This structure separates operator intention from the final physical output and provides defined points for troubleshooting and diagnostics.

---

## PLC Platform

| Item | Implementation |
|---|---|
| PLC | Allen-Bradley MicroLogix 1100 |
| Programming Software | RSLogix Micro Starter Lite 8.30 |
| Programming Language | Ladder Logic |
| Startup Delay | 5 seconds |
| Feedback Validation | 3 seconds |
| Runtime Simulation Interval | 10 seconds |

---

## Physical Inputs

| Address | Tag | Function |
|---|---|---|
| I:1/0 | Start_PB | Automatic start push button |
| I:1/1 | Stop_OK | Stop circuit healthy |
| I:1/2 | EStop_OK | Emergency-stop circuit healthy |
| I:1/3 | Overload_OK | Motor overload circuit healthy |
| I:1/4 | Auto_Mode | Automatic mode selector |
| I:1/5 | Manual_Mode | Manual mode selector |
| I:1/6 | Manual_Run_PB | Manual hold-to-run command |
| I:1/7 | Reset_PB | Master-fault reset |
| I:1/8 | Motor_Feedback | Motor/contactor feedback |

## Physical Outputs

| Address | Tag | Function |
|---|---|---|
| O:0/0 | Motor_Contactor | Motor starter/contactor command |
| O:0/1 | Running_Light | Commanded-running indication |
| O:0/2 | Fault_Light | Master-fault indication |
| O:0/3 | Alarm_Horn | Audible fault indication |

---

## Operating Sequence

### Automatic Mode

1. The PLC verifies the protective inputs and operating-mode selection.
2. Exactly one operating mode must be selected.
3. `Start_Permissive` becomes true when the required conditions are healthy.
4. `System_Ready` indicates that startup conditions are satisfied.
5. Pressing `Start_PB` in Auto mode latches `Auto_Run_Request`.
6. The request passes into the common `Start_Request`.
7. A 5-second startup timer begins.
8. When the timer completes and no master fault exists, `Run_Command` energizes.
9. `Run_Command` energizes `Motor_Contactor`.
10. Motor feedback is checked after a 3-second validation interval.

### Manual Mode

Manual operation uses a non-latched `Mode_Run_Request`.

The operator must hold `Manual_Run_PB` while Manual mode and `System_Ready` are active. The Manual request then uses the same startup-delay, run-command, output, feedback, and diagnostic architecture as Auto mode.

---

## Permissives and Interlocks

Startup requires:

- E-stop circuit healthy
- Stop circuit healthy
- Motor overload circuit healthy
- No active master fault
- Exactly one valid operating mode

Selecting Auto and Manual simultaneously creates `Mode_Fault`, preventing normal startup and contributing to the master-fault response.

---

## Feedback and Fault Handling

After `Motor_Contactor` is commanded ON, `Feedback_Timer T4:1` provides a 3-second validation interval.

If `Motor_Feedback I:1/8` is absent after the timer completes:

`Feedback_Fault → Fault_Active → Run_Command OFF`

The master fault also activates:

- `Fault_Light O:0/2`
- `Alarm_Horn O:0/3`

The master fault remains latched until the active fault conditions are restored and a valid Reset command is accepted.

---

## Production / Maintenance Monitoring

### Completed Start Events

`T4:0/DN → ONS B3:2/0 → CTU C5:0`

The one-shot ensures that the counter increments only once for each completed startup-delay event.

### Runtime Accumulation

`Motor_Contactor → T4:2 → ONS B3:2/1 → ADD N7:0 → RES T4:2`

During simulation, each completed 10-second runtime interval increments `N7:0`.

---

## Testing & Validation

The final PLC program was functionally tested for:

- System-ready conditions
- Automatic startup
- 5-second startup delay
- Manual hold-to-run operation
- Normal stopping
- Auto/Manual mode conflict
- E-stop monitoring
- Overload monitoring
- Motor feedback failure
- Master-fault latching
- Fault reset and recovery
- Completed-start event counting
- Runtime accumulation

**14 defined functional tests passed.**

Testing was performed in the PLC development/simulation environment and represents functional PLC program validation rather than physical machine commissioning or safety-system validation.

---

## Engineering Lessons

This project demonstrates the progression from a simple timer-controlled motor exercise to a more structured controls architecture.

Key engineering concepts demonstrated include:

- Separating operator requests from physical outputs
- Centralizing startup permissives
- Using different request behavior for Auto and Manual modes
- Separating commanded operation from verified equipment response
- Using one-shots for scan-based event processing
- Designing a traceable troubleshooting path
- Using latched faults for deliberate recovery

---

## Industrial Relevance

The control concepts demonstrated in this project are applicable to industrial equipment such as:

- Motors
- Pumps
- Fans
- Conveyors
- Mixers
- Packaging equipment
- Material-handling systems

The project is a training and portfolio implementation. In a production system, personnel-safety functions such as emergency stopping would require appropriately engineered safety-rated hardware, wiring, and validation.

---

## Documentation

Detailed engineering documentation includes:

- Project requirements
- I/O assignments
- Control strategy
- Engineering design decisions
- Complete rung-by-rung ladder explanation
- Sequence of operation
- Testing and validation
- Troubleshooting guidance
- Lessons learned
- Version 1 → Version 2 engineering progression
