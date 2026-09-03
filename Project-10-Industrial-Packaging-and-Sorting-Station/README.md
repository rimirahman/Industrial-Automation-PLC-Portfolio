# Project 10 — Industrial Packaging and Sorting Station

## Project Overview

This project is the final PLC project in my industrial automation portfolio and develops a packaging and sorting application into a structured PLC-controlled production station.

The system transports packages on a conveyor, counts production, identifies packages requiring rejection, operates a reject diverter, verifies the physical reject result, detects abnormal machine conditions, supports Manual/Maintenance operation, monitors production quality, and stops automatic production when a ten-package batch is complete.

The control architecture separates operator commands, machine states, operating modes, permissives, physical outputs, process feedback, verification, diagnostics, and production information rather than allowing field inputs to directly control outputs.

The project was developed and functionally tested in a PLC/simulation environment using an Allen-Bradley MicroLogix PLC and RSLogix Micro Starter Lite.

---

## Project Files

- 📄 [Full Engineering Report](./Project_10_Industrial_Packaging_and_Sorting_Station.pdf)
- ⚙️ [RSLogix 500 PLC Program](./PV10.RSS)

> The `.RSS` file contains the native RSLogix 500 ladder-logic project. RSLogix 500 or compatible Rockwell Automation software is required to open the PLC program.

---

## Key Features

- Latched Run Request
- System Ready machine state
- Safety-loss restart prevention
- Mutually exclusive Automatic and Manual modes
- Automatic Conveyor Permissive
- Common Auto/Manual conveyor output
- Common Auto/Manual reject-diverter output
- Ten-package production batch
- Retained reject-cycle state
- Timed reject diverter operation
- Physical reject confirmation
- Verified reject counting
- Reject-cycle timeout monitoring
- Reject-verification timeout monitoring
- Package photoeye stuck detection
- Reject sensor stuck detection
- Latched master fault
- Controlled Reset and separate restart
- Batch-complete buzzer and lamp
- High-reject-rate quality warning
- Total, reject, and good-package production statistics
- Manual conveyor and diverter hold-to-run control

---

## Control Architecture

The final control path is:

`Inputs → Operating Mode → Machine State → Automatic / Manual Command → Common Protection → Physical Output → Process Feedback → Verification / Diagnostics → Production Information`

The reject sequence follows:

`Reject Detection → Reject Cycle Active → Conveyor Paused → Diverter Timed → Reject Confirmation → Reject Verified → Reject Count → Reject Cycle Cleared`

This structure separates a PLC command from the verified physical process result.

---

## PLC Platform

| Item | Implementation |
|---|---|
| PLC | Allen-Bradley MicroLogix |
| Programming Software | RSLogix Micro Starter Lite |
| Programming Language | Ladder Logic |
| Reject Diverter Timer | T4:0 — 2 s |
| Reject Cycle Timeout | T4:1 — 5 s |
| Package Sensor Stuck Timer | T4:2 — 5 s |
| Reject Sensor Stuck Timer | T4:3 — 5 s |
| Reject Verification Timer | T4:4 — 5 s |
| Total Package Counter | C5:0 — PRE 10 |
| Reject Package Counter | C5:1 |
| Total Packages | N7:0 |
| Total Rejects | N7:1 |
| Good Packages | N7:2 |

---

## Physical Inputs

| Address | Function |
|---|---|
| I:1/0 | Start Push Button |
| I:1/1 | Stop Push Button |
| I:1/2 | Package Photoeye |
| I:1/3 | Reject Sensor |
| I:1/4 | Reset Push Button |
| I:1/5 | Safety OK |
| I:1/6 | Reject Confirm Sensor |
| I:1/7 | Auto Mode Select |
| I:1/8 | Manual Mode Select |
| I:1/9 | Manual Conveyor PB |
| I:1/10 | Manual Diverter PB |

---

## Physical Outputs

| Address | Function |
|---|---|
| O:2/0 | Conveyor Motor |
| O:2/1 | Reject Diverter |
| O:2/2 | Batch Complete Buzzer |
| O:2/3 | System Running Lamp |
| O:2/4 | Batch Complete Lamp |
| O:2/5 | Fault Indicator |
| O:2/6 | Quality Alarm Lamp |

---

## Run Control

Pressing Start latches:

`Run_Request B3:0/0`

The stored run request is removed by:

- Stop Push Button
- Fault Active
- Loss of Safety OK

This means restoration of Safety OK alone does not automatically restart the conveyor.

A new Start command is required after recovery.

---

## System Ready

`System_Ready B3:0/1`

requires:

- Run Request active
- Safety OK
- No active master fault

This separates operator intent from current PLC authorization.

---

## Auto / Manual Mode Ownership

Automatic Mode becomes active only when:

- Auto selector is ON
- Manual selector is OFF

Manual Mode becomes active only when:

- Manual selector is ON
- Auto selector is OFF

This prevents ambiguous command ownership.

---

## Automatic Conveyor Control

Automatic conveyor operation requires:

- System Ready
- Auto Mode active
- Batch Complete OFF
- Reject Cycle Active OFF

These conditions establish:

`Conveyor_Permissive B3:0/2`

The final Conveyor Motor rung accepts either:

- Automatic Conveyor Permissive
- Manual Conveyor Command

but both paths remain subject to:

- Safety OK
- No Fault Active

This provides one physical output ownership point for the conveyor.

---

## Package Counting

`C5:0 Total_Package_Count`

counts package-photoeye transitions only during valid Auto Mode production.

The final preset is:

`10 packages`

Manual conveyor testing therefore does not alter the production count.

When C5:0 reaches its preset:

`Batch_Complete B3:1/1`

is latched.

---

## Reject Sequence

A reject detection during Auto Mode latches:

`Reject_Cycle_Active B3:1/0`

Once active, the reject cycle continues independently of the original reject-sensor signal.

Reject Cycle Active also removes the Automatic Conveyor Permissive so package movement pauses during reject handling.

---

## Reject Diverter Timing

`T4:0 Reject_Diverter_Timer`

uses:

- Time base: 1.0 s
- Preset: 2

During the automatic reject cycle, the diverter remains energized until the timer reaches Done.

Manual Diverter operation provides an alternate command path to the same final physical output.

Both Automatic and Manual diverter operation require:

- Safety OK
- No active master fault

---

## Reject Verification

The PLC does not count a reject simply because the diverter was commanded.

During an active reject cycle:

`Reject_Confirm_Sensor I:1/6`

latches:

`Reject_Verified B3:1/2`

Only the verified event increments:

`C5:1 Reject_Package_Count`

This keeps reject statistics tied to confirmed process results rather than actuator commands.

---

## Reject-Cycle Supervision

Two different timers supervise different reject failure conditions.

### Reject Cycle Timeout

`T4:1`

monitors total reject-cycle duration.

If the reject cycle remains active too long:

`Reject_Timeout B3:2/0`

becomes active.

### Reject Verification Timeout

`T4:4`

monitors how long the PLC waits for physical reject confirmation.

If confirmation does not arrive in time:

`Reject_Verification_Fault B3:2/3`

becomes active.

These diagnostics intentionally answer different questions.

---

## Sensor Diagnostics

The project monitors continuously active field sensors.

### Package Photoeye

`T4:2`

detects the Package Photoeye remaining continuously ON for 5 seconds.

This generates:

`Package_Sensor_Fault B3:2/1`

### Reject Sensor

`T4:3`

detects the Reject Sensor remaining continuously ON for 5 seconds.

This generates:

`Reject_Sensor_Fault B3:2/2`

These diagnostics help distinguish normal package detection from abnormal blocked or stuck sensor conditions.

---

## Master Fault Handling

Any of the following can latch:

`Fault_Active B3:0/3`

- Reject Timeout
- Package Sensor Fault
- Reject Sensor Fault
- Reject Verification Fault

Once Fault Active is latched:

- Run Request is removed
- System Ready drops
- Automatic operation stops
- Manual outputs are also inhibited through the common protection layer
- Fault Indicator energizes

The individual diagnostic bits remain available to identify the cause.

---

## Controlled Reset & Restart

Reset clears applicable:

- Master fault
- Run Request
- Batch Complete
- Reject Cycle Active
- Total Package Counter
- Reject Package Counter

Reset does not restart the machine.

A separate Start command is required for the next operating cycle.

This prevents unintended automatic restart following fault or safety recovery.

---

## Batch Completion

When the total package counter reaches 10:

`Batch_Complete B3:1/1`

latches.

Batch Complete:

- Stops automatic conveyor production
- Energizes the Batch Complete Buzzer
- Energizes the Batch Complete Lamp

Batch completion is treated as a planned production state rather than a machine fault.

---

## Quality Monitoring

The PLC evaluates:

`C5:1.ACC >= 3`

If three or more verified rejects are present:

`High_Reject_Rate B3:2/4`

becomes active and energizes:

`Quality_Alarm_Lamp O:2/6`

This condition is intentionally a **quality warning**, not a machine-stopping fault.

Production can continue until another condition, such as Batch Complete, stops the process.

---

## Manual / Maintenance Mode

Manual Mode provides hold-to-run operation for:

- Conveyor
- Reject Diverter

The manual pushbuttons generate internal command bits rather than directly energizing physical outputs.

The final output rungs still require:

- Safety OK
- No active Fault

Manual operation therefore changes the command source but does not bypass the common machine protection layer.

---

## Production Statistics

The program exposes production values through integer registers.

### Total Packages

`N7:0 = C5:0.ACC`

### Total Rejects

`N7:1 = C5:1.ACC`

### Good Packages

`N7:2 = N7:0 - N7:1`

This produces simple production values that can later be displayed through an HMI or SCADA system.

---

## Testing & Validation

The final PLC program was functionally validated for:

- Initial Reset
- Auto/Manual mode selection
- Automatic startup
- Package counting
- Successful reject handling
- Verified reject counting
- Reject verification failure
- Reject cycle timeout
- Package sensor stuck fault
- Reject sensor stuck fault
- Master fault shutdown
- Fault Reset
- Safety loss
- Safety recovery
- Manual conveyor
- Manual diverter
- Manual protection
- Manual data integrity
- High-reject-rate quality warning
- Production statistics
- Batch completion
- Batch Reset
- Integrated production sequence
- Integrated fault recovery

**All documented validation tests passed.**

The final integrated batch produced:

- **10 Total Packages**
- **3 Verified Rejects**
- **7 Good Packages**

At that condition:

- Batch Complete was active
- Quality Warning was active
- Fault Active remained OFF

Testing was performed in the PLC/simulation environment and does not represent physical machine commissioning or personnel-safety certification.

---

## Engineering Lessons

This project demonstrates several important controls-engineering concepts:

- Separating operator commands from physical outputs
- Machine-state architecture
- Automatic and Manual command ownership
- Common equipment permissives
- Single physical output ownership
- Retained sequence states
- Timed actuator control
- Physical process verification
- Command vs. confirmed result
- Sensor supervision
- Process timeout diagnostics
- Latched master faults
- Controlled recovery
- Restart prevention
- Planned production-state handling
- Quality warnings separate from machine faults
- Production statistics
- Structured PLC troubleshooting
- Future HMI-ready data organization

---

## Industrial Relevance

The architecture demonstrated in this project is applicable to:

- Packaging lines
- Inspection stations
- Sorting cells
- Material-handling systems
- Assembly lines
- Automated production systems

The strongest engineering concept is the complete path:

`Inputs → Machine States → Commands → Permissives → Outputs → Feedback → Verification → Diagnostics → Production Information`

A larger industrial implementation could additionally include actuator-position feedback, motor or drive status, configurable batch sizes, configurable quality thresholds, HMI alarms, production orders, defect classifications, SCADA/historian integration, recipes, and safety-rated hardware.
