# Project 06 — Industrial Conveyor Sorting System

## Project Overview

This project develops a basic conveyor application into a structured industrial conveyor sorting system using an Allen-Bradley MicroLogix 1100 PLC.

The system transports products between stations, retains product classification, automatically handles rejected products, verifies successful rejection, detects abnormal process conditions, and maintains production statistics.

The project was developed and functionally tested in a PLC training/simulation environment using RSLogix Micro Starter Lite.

---

## Project Files

- 📄 [Full Engineering Report](./Project_06_Industrial_Conveyor_Sorting_System.pdf)
- ⚙️ [RSLogix 500 PLC Program](./PV6.RSS)

> The `.RSS` file contains the native RSLogix 500 ladder-logic project. RSLogix 500 or compatible Rockwell Automation software is required to open the PLC program.

---

## Key Features

- Auto and Manual operating modes
- Auto/Manual mode-conflict detection
- E-stop monitoring
- Conveyor overload monitoring
- Pneumatic air-pressure monitoring
- Latched master fault
- Process-generated fault handling
- Controlled fault reset
- Sequence recovery reset
- Centralized Start Permissive
- Latched Machine Run Request
- System Ready status
- Separate automatic and manual conveyor requests
- 3-second conveyor startup delay
- Product tracking between stations
- Stored reject classification
- Product-travel jam monitoring
- Automatic reject sequencing
- Manual reject operation
- Timed reject actuator control
- Reject confirmation
- Reject timeout detection
- Total product counting
- Separate accepted and rejected production totals
- Controlled state cleanup after processing

---

## Control Architecture

The program follows the control path:

`Machine Conditions → Start Permission → Run Request → System Ready → Conveyor Control → Product Tracking → Classification → Accepted/Reject Processing → Counting → Cleanup`

This architecture separates:

- Machine authorization
- Operator commands
- Physical outputs
- Product tracking
- Product classification
- Reject sequencing
- Process verification
- Fault diagnostics
- Production statistics

This makes the PLC program easier to monitor and troubleshoot than direct sensor-to-output control.

---

## PLC Platform

| Item | Implementation |
|---|---|
| PLC | Allen-Bradley MicroLogix 1100 |
| Programming Software | RSLogix Micro Starter Lite |
| Programming Language | Ladder Logic |
| Conveyor Startup Timer | T4:0 — 3 seconds |
| Jam Timer | T4:1 — 120 seconds |
| Reject Extend Timer | T4:2 — 10 seconds |
| Total Product Counter | C5:0 |
| Accepted Product Total | N7:0 |
| Rejected Product Total | N7:1 |

---

## Physical Inputs

| Address | Tag | Function |
|---|---|---|
| I:1/0 | Start_PB | Machine start command |
| I:1/1 | Stop_PB | Machine stop command |
| I:1/2 | Reset_PB | Fault and sequence recovery |
| I:1/3 | Auto_Mode | Automatic production mode |
| I:1/4 | Manual_Mode | Manual maintenance mode |
| I:1/5 | EStop_OK | E-stop healthy feedback |
| I:1/6 | Overload_OK | Conveyor overload healthy feedback |
| I:1/7 | Entry_Photoeye | Product entry detection |
| I:1/8 | Inspection_Reject | Reject classification input |
| I:1/9 | Sort_Photoeye | Sorting-station product detection |
| I:1/10 | Reject_Confirm | Reject confirmation feedback |
| I:1/11 | Air_Pressure_OK | Pneumatic utility healthy feedback |
| I:1/12 | Manual_Conveyor_PB | Hold-to-run manual conveyor |
| I:1/13 | Manual_Reject_PB | Hold-to-run manual reject |

## Physical Outputs

| Address | Tag | Function |
|---|---|---|
| O:2/0 | Conveyor_Motor | Conveyor motor command |
| O:2/1 | Reject_Solenoid | Pneumatic reject command |

---

## Important Internal PLC Data

| Address | Tag | Purpose |
|---|---|---|
| B3:0/0 | Machine_Run_Request | Latched operator run request |
| B3:0/1 | Start_Permissive | Machine operating permission |
| B3:0/2 | System_Ready | Ready/running state |
| B3:0/3 | Mode_Conflict | Auto/Manual conflict diagnostic |
| B3:0/4 | Fault_Active | Latched master fault |
| B3:0/5 | Conveyor_Request | Automatic conveyor request |
| B3:0/6 | Product_Tracked | Stored active-product state |
| B3:0/7 | Reject_Product | Stored reject classification |
| B3:1/0 | Reject_Sequence | Automatic reject sequence |
| B3:1/1 | Jam_Fault | Latched product-travel fault |
| B3:1/2 | Reject_Timeout_Fault | Missing reject-confirmation fault |
| B3:1/3 | Manual_Conveyor_Request | Manual conveyor request |
| B3:1/4 | Manual_Reject_Request | Manual reject request |
| B3:1/5 | Accepted_Event | Accepted-product completion event |
| B3:1/6 | Reject_Complete_Event | Confirmed reject completion event |

---

## Machine Authorization

Machine operation begins with evaluation of:

- Auto/Manual mode selection
- E-stop healthy feedback
- Conveyor overload feedback
- Pneumatic air-pressure feedback
- Stored process faults

Invalid conditions can latch `Fault_Active B3:0/4`.

Healthy operating conditions establish:

`Start_Permissive B3:0/1`

Pressing Start while the permissive is valid latches:

`Machine_Run_Request B3:0/0`

System Ready then requires:

`Start Permissive + Machine Run Request + Auto or Manual Mode`

This keeps operator demand separate from current machine authorization.

---

## Conveyor Control

Automatic and manual conveyor commands are maintained separately.

### Automatic Mode

`System Ready + Auto Mode + NOT Reject Sequence → Conveyor Request`

### Manual Mode

`System Ready + Manual Mode + Manual Conveyor PB → Manual Conveyor Request`

Either conveyor request starts `T4:0`.

The timer uses:

- Time base: 1.0 second
- Preset: 3
- Startup delay: **3 seconds**

The physical conveyor motor energizes only when:

`T4:0/DN + Active Conveyor Request → Conveyor Motor`

This ensures the startup timer does not become a permanent motor command.

---

## Product Tracking

When an incoming product activates `Entry_Photoeye I:1/7` during valid automatic operation:

`Product_Tracked B3:0/6`

is latched.

This allows the PLC to remember the product after it physically leaves the entry photoeye.

The logic blocks another product-entry event while a product is already being tracked.

---

## Reject Classification

While the current product is being tracked at the entry station:

`Inspection_Reject I:1/8`

can latch:

`Reject_Product B3:0/7`

The classification therefore remains stored after the product leaves the inspection sensor.

This demonstrates an important industrial concept:

**The PLC retains product information independently from the instantaneous field input that originally generated it.**

---

## Product Travel Monitoring

`Jam_Timer T4:1` monitors the tracked product while:

- Product Tracked is true
- Sort Photoeye has not detected the product
- Conveyor Motor is running

Timer configuration:

- Time base: 1.0 second
- Preset: 120
- Programmed interval: **120 seconds**

If the timer reaches Done while the original abnormal travel conditions still exist:

`Jam_Fault B3:1/1`

is latched.

The conditions are rechecked when the fault is generated so the fault represents an unresolved product-travel condition rather than only a timer state.

---

## Accepted Product Processing

When:

`Product Tracked + Sort Photoeye + NOT Reject Product`

are true, the PLC generates:

`Accepted_Event B3:1/5`

The accepted event is used for:

- Total product counting
- Accepted-product counting
- Product-state cleanup

This means the product is counted after reaching a completed sorting result rather than merely when entering the conveyor.

---

## Automatic Reject Sequence

A reject product reaching the sorting station causes:

`Reject_Sequence B3:1/0`

to latch.

The reject sequence also removes the automatic Conveyor Request, allowing product movement to stop while rejection is performed.

`Reject_Extend_Timer T4:2` then controls the reject interval.

Timer configuration:

- Time base: 0.01 second
- Preset: 1000
- Programmed interval: **10 seconds**

During the automatic reject interval:

`Reject Sequence + NOT T4:2/DN + NOT Fault Active → Reject Solenoid`

---

## Manual Reject Control

Manual reject operation requires:

`System Ready + Manual Mode + Manual Reject PB + NOT Fault Active`

This generates:

`Manual_Reject_Request B3:1/4`

Manual operation is hold-to-run.

Automatic and manual reject commands both control the same physical `Reject_Solenoid O:2/1` rung, avoiding duplicate output coils.

---

## Reject Verification

The PLC does not assume that energizing the reject solenoid means the product was successfully rejected.

Successful completion requires:

`Reject Sequence + T4:2/DN + Reject Confirm`

which generates:

`Reject_Complete_Event B3:1/6`

This separates:

**Actuator Command**

from

**Process Verification**

and demonstrates closed-loop sequence confirmation.

---

## Reject Timeout Fault

If the reject timer completes but `Reject_Confirm I:1/10` is still absent:

`Reject_Timeout_Fault B3:1/2`

is latched.

The timeout fault feeds the master fault architecture and prevents an unsuccessful reject attempt from being treated as completed production.

---

## Production Counting

The project maintains three production values.

### Total Products

`Accepted Event OR Reject Complete Event → CTU C5:0`

### Accepted Products

`Accepted Event → ADD 1 to N7:0`

### Rejected Products

`Reject Complete Event → ADD 1 to N7:1`

Rejected products are therefore counted only after successful reject confirmation.

This gives the PLC:

- Total processed products
- Accepted products
- Successfully rejected products

---

## Sequence Cleanup

Cleanup occurs after the counting rungs.

For an accepted product:

- Product Tracked is cleared
- Reject Product is cleared

For a confirmed reject:

- Reject Sequence is cleared
- Product Tracked is cleared
- Reject Product is cleared

Placing counting before cleanup deliberately uses PLC scan order so production totals are updated before the stored product states are removed.

---

## Fault Handling

The master fault architecture monitors:

- Auto/Manual mode conflict
- E-stop feedback loss
- Conveyor overload feedback loss
- Air-pressure feedback loss
- Product travel jam
- Reject confirmation timeout

`Fault_Active B3:0/4` is latched so temporary abnormal conditions require deliberate operator recovery.

Fault reset requires the implemented healthy-condition chain to be restored before Reset can clear:

- Fault Active
- Jam Fault
- Reject Timeout Fault

A separate sequence-recovery rung clears abandoned:

- Reject Sequence
- Product Tracked
- Reject Product

states during controlled recovery.

---

## Testing & Validation

The final PLC implementation was validated for:

- Ready-state generation
- Delayed conveyor startup
- Stop behavior
- E-stop monitoring
- Overload monitoring
- Air-pressure monitoring
- Auto/Manual mode conflict
- Controlled fault recovery
- Good-product tracking
- Accepted-product counting
- Reject-product classification
- Automatic reject sequence
- Reject confirmation
- Rejected-product counting
- Jam detection
- Reject timeout detection
- Manual conveyor operation
- Manual reject operation

**All documented validation categories passed.**

Testing represents PLC/simulation validation and not physical conveyor commissioning or machine-safety certification.

---

## Engineering Lessons

This project demonstrates several important industrial controls concepts:

- State-based product tracking
- Retaining information between physical sensing locations
- Separating inspection inputs from stored product classification
- Auto/Manual machine architecture
- Hold-to-run manual control
- Separating sequence requests from physical outputs
- Controlled equipment startup
- Process travel monitoring
- Automatic reject sequencing
- Distinguishing actuator command from process confirmation
- Detecting unsuccessful sequence completion
- Latched fault handling
- Controlled recovery
- Production statistics
- Deliberate use of PLC scan order
- State cleanup after completed processing

---

## Version 1 → Version 2 Development

The original Version 1 project established the foundational conveyor and product-sorting sequence. Version 2 develops that foundation into a more structured industrial conveyor sorting system with improved product tracking, reject handling, verification, and diagnostics.

- 📄 [View Original Version 1 Project](./Version-1/Project_06_Version_1.pdf)

Version 2 expands the original application with Auto/Manual operating modes, centralized permissives, System Ready logic, retained product tracking and classification, automatic reject sequencing, physical reject confirmation, production counting, conveyor jam monitoring, master-fault handling, and controlled recovery.

---

## Industrial Relevance

Industrial conveyor sorting systems are used in:

- Packaging
- Manufacturing
- Logistics
- Warehousing
- Food processing
- Automated material handling

The project demonstrates product tracking between stations, retained classification, Auto/Manual operation, machine permissives, process timers, reject sequencing, actuator confirmation, jam detection, production counting, and controlled fault recovery.

A physical implementation could additionally include VFD control, motor-running feedback, safety-rated devices, pneumatic instrumentation, encoder-based positioning, multiple-product tracking, barcode or vision inspection, HMI alarm history, and production data logging.
