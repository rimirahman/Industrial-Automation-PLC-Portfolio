# Project 05 — Industrial Bottle Filling and Capping System

## Project Overview

This project develops a conveyor-based application into a structured industrial bottle filling and capping system using an Allen-Bradley MicroLogix 1100 PLC.

The system combines machine start/stop control, operating permissives, bottle detection, conveyor sequencing, timed filling-stage control, timed capping, production counting, fault handling, and machine-status indication.

The project was developed and functionally tested in a PLC training/simulation environment using RSLogix Micro Starter Lite.

---

## Project Files

- 📄 [Full Engineering Report](./Project_05_Industrial_Bottle_Filling_and_Capping_System.pdf)
- ⚙️ [RSLogix 500 PLC Program](./PV5.RSS)

> The `.RSS` file contains the native RSLogix 500 ladder-logic project. RSLogix 500 or compatible Rockwell Automation software is required to open the PLC program.

---

## Key Features

- Auto and Manual operating modes
- Auto/Manual mode-conflict detection
- E-stop and overload monitoring
- Latched master fault
- Conditional fault reset
- Centralized Start Permissive
- Latched Machine Run Request
- System Ready authorization
- Internal Conveyor Request
- Bottle photoeye detection
- Latched Filling Active state
- 3-second filling-stage interval
- Explicit filling-to-capping transition
- Latched Capping Active state
- 2-second capping interval
- Cap solenoid control
- Automatic conveyor restart
- Completed-bottle production counting
- Running, fault, and audible alarm indication
- Operator production-counter reset

---

## Control Architecture

The sequence is structured as:

`Machine Health → Run Authorization → Conveyor Request → Filling State → Capping State → Completed Production`

The operator Start command does not directly control the conveyor.

Instead:

`Start PB + Start Permissive → Machine Run Request → System Ready`

Automatic production then follows:

`Conveyor → Bottle Detection → Filling → Capping → Production Count → Conveyor Restart`

This separates machine authorization, sequence states, physical outputs, and production information.

---

## PLC Platform

| Item | Implementation |
|---|---|
| PLC | Allen-Bradley MicroLogix 1100 |
| Programming Software | RSLogix Micro Starter Lite |
| Programming Language | Ladder Logic |
| Filling Timer | T4:1 — 3 seconds |
| Capping Timer | T4:2 — 2 seconds |
| Production Counter | C5:0 — PRE 9999 |

---

## Physical Inputs

| Address | Tag | Function |
|---|---|---|
| I:1/0 | Start_PB | Machine Start command |
| I:1/1 | Stop_PB | Machine Stop command |
| I:1/2 | Reset_PB | Fault/counter reset |
| I:1/3 | Auto_Mode | Automatic mode selection |
| I:1/4 | Manual_Mode | Manual mode selection |
| I:1/5 | EStop_OK | E-stop healthy feedback |
| I:1/6 | Overload_OK | Conveyor overload healthy feedback |
| I:1/7 | Bottle_Photoeye | Bottle detection at station |

## Physical Outputs

| Address | Tag | Function |
|---|---|---|
| O:2/0 | Conveyor_Motor | Conveyor motor command |
| O:2/1 | Cooling_Fan | Filling-interval output |
| O:2/2 | Cap_Solenoid | Capping actuator command |
| O:2/3 | Running_Lamp | System Ready indication |
| O:2/4 | Fault_Lamp | Master fault indication |
| O:2/5 | Alarm_Horn | Audible fault indication |

---

## Important Internal States

| Address | Tag | Purpose |
|---|---|---|
| B3:0/2 | Conveyor_Request | Requests conveyor movement |
| B3:0/3 | System_Ready | Machine authorization status |
| B3:0/4 | Fault_Active | Latched master fault |
| B3:0/5 | Mode_Conflict | Invalid Auto/Manual selection |
| B3:0/6 | Filling_Active | Filling process state |
| B3:0/7 | Start_Permissive | Combined start permission |
| B3:1/0 | Capping_Active | Capping process state |
| B3:1/2 | Machine_Run_Request | Stored operator run request |

---

## Machine Authorization

Machine operation begins with the health and authorization logic.

`Start_Permissive` requires:

- No active master fault
- E-stop healthy
- Overload healthy
- No Auto/Manual mode conflict

Pressing Start while the permissive is valid latches:

`Machine_Run_Request B3:1/2`

System Ready then requires:

`Start Permissive + Machine Run Request + Valid Operating Mode`

This keeps machine authorization separate from individual physical outputs.

---

## Automatic Conveyor Sequence

In Auto mode, when:

- System Ready is true
- Bottle Photoeye is clear
- Filling is inactive
- Capping is inactive

the PLC latches:

`Conveyor_Request B3:0/2`

The physical conveyor motor requires:

`Conveyor Request + System Ready + No Fault → Conveyor Motor`

This keeps the physical motor behind both the sequence request and machine authorization.

---

## Bottle Detection and Filling

When the bottle reaches the station:

`Conveyor Request + Bottle Photoeye + System Ready → Filling Active`

Bottle detection also unlatches Conveyor Request, stopping the conveyor at the processing position.

`Filling_Active` then enables `Fill_Delay T4:1`.

The timer uses:

- Time base: 0.01 s
- Preset: 300
- Programmed interval: **3 seconds**

During this interval, the final screenshot-labeled `Cooling_Fan O:2/1` output is energized while Filling Active and System Ready remain true and the timer has not completed.

---

## Filling-to-Capping Transition

When `T4:1/DN` becomes true:

1. `Filling_Active` is unlatched.
2. `Capping_Active` is latched.

Separate transition rungs make the process-state change explicit and easier to monitor during troubleshooting.

---

## Capping Sequence

`Capping_Active B3:1/0` starts `Cap_Timer T4:2`.

The timer uses:

- Time base: 0.01 s
- Preset: 200
- Programmed interval: **2 seconds**

During the capping interval:

`Capping Active + NOT Cap Timer Done → Cap Solenoid O:2/2`

When the timer completes:

- Capping Active is unlatched.
- Conveyor Request is relatched.

The completed bottle can then leave the processing station.

---

## Production Counting

`Bottle_Counter C5:0` counts completed capping cycles.

The counter is triggered from:

`T4:2/DN → CTU C5:0`

Counting at the end of the capping process represents completed production rather than merely detecting an incoming bottle.

The counter preset is:

`9999`

`Reset_PB I:1/2` executes `RES C5:0` when the production count needs to be cleared.

---

## Fault Handling

`Fault_Active B3:0/4` can latch from:

- Auto/Manual Mode Conflict
- Loss of EStop_OK
- Loss of Overload_OK

A fault removes the stored Machine Run Request and prevents normal machine authorization.

Fault reset is allowed only when:

- Reset PB is active
- E-stop feedback is healthy
- Overload feedback is healthy
- Mode Conflict is false

This prevents the machine from automatically recovering while an abnormal monitored condition remains present.

---

## Operator Indication

The project provides:

- `Running_Lamp O:2/3` — follows System Ready
- `Fault_Lamp O:2/4` — follows Fault Active
- `Alarm_Horn O:2/5` — follows Fault Active

Using System Ready for the Running Lamp allows the machine to remain indicated as operational even while the conveyor is intentionally stopped during filling or capping.

---

## Testing & Validation

The final PLC program was functionally tested for:

- Start Permissive
- Machine Run Request
- Automatic conveyor startup
- Bottle detection
- Conveyor stopping at the processing station
- 3-second filling-stage timing
- Filling-interval output
- Filling-to-capping state transition
- 2-second capping timing
- Cap solenoid operation
- Conveyor restart after capping
- Completed-bottle counting
- Stop behavior
- E-stop fault response
- Overload fault response
- Auto/Manual mode conflict
- Controlled fault reset
- Production-counter reset

**17 defined functional tests passed.**

Testing represents PLC program validation in the development/simulation environment rather than physical packaging-machine commissioning or safety-system validation.

---

## Engineering Lessons

This project demonstrates several important controls-engineering concepts:

- Separating operator commands from equipment outputs
- Using internal request bits for sequence control
- Maintaining process states independently from sensor events
- Designing sequential machine logic with explicit state transitions
- Using separate timers for independent process stages
- Keeping physical outputs behind machine authorization
- Stopping and restarting conveyors around processing stations
- Counting production at meaningful process milestones
- Centralizing fault handling and controlled recovery
- Structuring ladder logic for online troubleshooting and maintainability

---

## Industrial Relevance

Bottle filling and capping sequences are widely used in:

- Packaging
- Food and beverage
- Pharmaceutical manufacturing
- Chemical processing
- Consumer-product manufacturing

The project demonstrates machine permissives, latched run requests, sensor-based indexing, timed process stages, state transitions, conveyor interlocks, fault handling, operator indication, and production counting.

A production system could additionally include bottle-position sensors, fill-level or flow feedback, cap-present verification, conveyor feedback, reject handling, guarding, safety-rated hardware, HMI functions, recipes, and additional process diagnostics.
