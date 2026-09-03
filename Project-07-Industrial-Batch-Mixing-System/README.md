# Project 07 — Industrial Batch Mixing System

## Project Overview

This project develops a basic batch-mixing application into a structured industrial PLC-controlled sequential process using an Allen-Bradley MicroLogix 1100 PLC.

The system automatically fills Material A, transfers to Material B filling, performs a timed mixing stage, drains the vessel, records a completed batch, and prepares the sequence for the next automatic cycle.

Manual Mode provides separate hold-to-run controls for Valve A, Valve B, the Mixer Motor, and the Drain Valve with equipment and process interlocks.

The project was developed and functionally tested in a PLC training/simulation environment using RSLogix Micro Starter Lite.

---

## Project Files

- 📄 [Full Engineering Report](./Project_07_version2-Industrial_Batch_Mixing_System.pdf)
- ⚙️ [RSLogix 500 PLC Program](./PV72.RSS)

> The `.RSS` file contains the native RSLogix 500 ladder-logic project. RSLogix 500 or compatible Rockwell Automation software is required to open the PLC program.

---

## Key Features

- Automatic and Manual operating modes
- Auto/Manual mode-conflict detection
- Tank level-consistency monitoring
- E-stop and mixer-overload monitoring
- Latched master fault handling
- Controlled fault reset and sequence recovery
- Centralized Start Permissive
- Latched Machine Run Request
- System Ready authorization
- Empty-tank batch-start verification
- Retained state-based batch sequencing
- Material A and Material B filling stages
- Independent filling timeout supervision
- Timed mixing stage
- Interlocked draining
- Automatic batch completion and counting
- Hold-to-run manual process controls
- Manual equipment mutual interlocks
- Single physical output coil per actuator
- Running, Ready, Fault, and Alarm indication

---

## Control Architecture

The PLC program follows the control structure:

`Machine Validation → Permission → Run Request → System Ready → Automatic Batch / Manual Request → Physical Output`

The automatic production sequence is:

`Fill Material A → Fill Material B → Mix → Drain → Batch Complete → Count → Cleanup → Next Batch`

Dedicated retained sequence bits represent the current process state rather than relying only on physical outputs or instantaneous sensor conditions.

---

## PLC Platform

| Item | Implementation |
|---|---|
| PLC | Allen-Bradley MicroLogix 1100 |
| Programming Software | RSLogix Micro Starter Lite |
| Programming Language | Ladder Logic |
| Fill A Timer | T4:0 — 120 seconds |
| Fill B Timer | T4:1 — 120 seconds |
| Mixing Timer | T4:2 — 10 seconds |
| Batch Counter | C5:0 — PRE 9999 |

---

## Physical Inputs

| Address | Tag | Function |
|---|---|---|
| I:1/0 | Start_PB | Machine start command |
| I:1/1 | Stop_PB | Machine stop command |
| I:1/2 | Reset_PB | Fault and sequence recovery |
| I:1/3 | Auto_Mode | Automatic mode selector |
| I:1/4 | Manual_Mode | Manual mode selector |
| I:1/5 | EStop_OK | E-stop healthy feedback |
| I:1/6 | Mixer_OL_OK | Mixer overload healthy feedback |
| I:1/7 | Low_Level_Switch | Low-level process feedback |
| I:1/8 | Material_A_Level | Intermediate level feedback |
| I:1/9 | High_Level_Switch | High-level/full feedback |
| I:1/12 | Manual_Drain_PB | Hold-to-run manual drain |
| I:1/13 | Manual_Valve_A_PB | Hold-to-run manual Valve A |
| I:1/14 | Manual_Valve_B_PB | Hold-to-run manual Valve B |
| I:1/15 | Manual_Mixer_PB | Hold-to-run manual mixer |

---

## Physical Outputs

| Address | Tag | Function |
|---|---|---|
| O:2/0 | Valve_A | Material A inlet command |
| O:2/1 | Valve_B | Material B inlet command |
| O:2/2 | Mixer_Motor | Mixer command |
| O:2/3 | Drain_Valve | Drain command |
| O:2/4 | Running_Lamp | Active-batch indication |
| O:2/5 | Ready_Lamp | Ready/idle indication |
| O:2/6 | Fault_Lamp | Master fault indication |
| O:2/7 | Alarm_Horn | Audible fault indication |

---

## Automatic Batch Sequence

### 1. Batch Start

A new automatic batch can begin only when:

- System Ready is established
- Auto Mode is selected
- No batch is already active
- Low Level is OFF
- Material A Level is OFF
- High Level is OFF

This confirms an empty vessel before `Batch_Active B3:0/5` is latched.

### 2. Fill Material A

`Fill_A_Step B3:0/6` opens `Valve_A O:2/0`.

Material A filling continues until:

`Material_A_Level I:1/8`

is reached.

`T4:0` supervises this operation with a maximum programmed interval of **120 seconds**.

If the target level is not reached in time, `Fill_Timeout_Fault B3:1/3` is latched.

### 3. Fill Material B

Reaching Material A Level transfers the sequence:

`Fill_A_Step → Fill_B_Step`

`Valve_B O:2/1` then fills the vessel until:

`High_Level_Switch I:1/9`

is reached.

`T4:1` independently supervises Fill B with another **120-second** timeout.

### 4. Mixing

High Level transfers the process:

`Fill_B_Step → Mix_Step`

`Mixer_Motor O:2/2` operates during the valid Mix Step.

`T4:2` provides a **10-second mixing interval**.

The timer runs from the actual mixer-output condition so the programmed interval represents commanded mixer operation.

### 5. Draining

When `T4:2/DN` becomes true:

`Mix_Step → Drain_Step`

`Drain_Valve O:2/3` operates only while the required interlocks are satisfied.

Automatic draining is prevented while Valve A, Valve B, or the Mixer Motor is operating.

### 6. Batch Completion

During Drain Step, when the Low Level Switch clears:

`Batch_Complete_Event B3:1/2`

is generated.

The completion event increments:

`Batch_Counter C5:0`

before the retained Drain Step and Batch Active states are cleared.

If Auto Mode, System Ready, and the empty-tank conditions remain valid, another automatic batch can then begin.

---

## Manual Operation

Manual Mode provides hold-to-run operation for:

- Valve A
- Valve B
- Mixer Motor
- Drain Valve

Manual commands do not directly bypass the process architecture.

Each pushbutton first generates a validated internal request:

- `B3:2/0` — Manual Valve A Request
- `B3:2/1` — Manual Valve B Request
- `B3:2/2` — Manual Mixer Request
- `B3:2/3` — Manual Drain Request

The request bits then feed the same physical output rungs used by the automatic sequence.

This maintains one physical OTE for each actuator.

---

## Manual Interlocks

Manual operation retains important equipment and process protections.

### Manual Valve A

Requires:

- System Ready
- Manual Mode
- Manual Valve A PB held
- No active fault
- Valve B OFF
- Drain Valve OFF
- High Level OFF

### Manual Valve B

Requires:

- System Ready
- Manual Mode
- Manual Valve B PB held
- No active fault
- Valve A OFF
- Drain Valve OFF
- High Level OFF

### Manual Mixer

Requires:

- System Ready
- Manual Mode
- Manual Mixer PB held
- No active fault
- Low Level ON
- Valve A OFF
- Valve B OFF
- Drain Valve OFF

The Low Level requirement prevents manual dry mixing.

### Manual Drain

Requires:

- System Ready
- Manual Mode
- Manual Drain PB held
- No active fault
- Valve A OFF
- Valve B OFF
- Mixer Motor OFF

---

## Fault Handling & Diagnostics

The PLC architecture monitors:

- Auto/Manual mode conflict
- E-stop healthy feedback
- Mixer overload healthy feedback
- Invalid tank-level combinations
- Material A filling timeout
- Material B filling timeout

Abnormal conditions can latch:

`Fault_Active B3:0/4`

A valid Reset requires the monitored machine conditions to return to their acceptable states before stored faults and sequence states can be cleared.

The program also provides a separate sequence-recovery reset for:

- Batch Active
- Fill A Step
- Fill B Step
- Mix Step
- Drain Step

This allows an interrupted batch to return to a known sequence state before another batch begins.

---

## Level Consistency Monitoring

The PLC checks the tank-level switches for physically inconsistent combinations.

Examples include:

- High Level ON while Material A Level is OFF
- Material A Level ON while Low Level is OFF

These conditions generate:

`Level_Conflict_Fault B3:1/5`

This helps detect inconsistent sensor, wiring, or simulated process conditions before they are used by the automatic sequence.

---

## Operator Indication

The system provides:

- **Running Lamp** — active automatic batch
- **Ready Lamp** — system enabled but no batch active
- **Fault Lamp** — master fault active
- **Alarm Horn** — audible master-fault indication

Ready and Running are intentionally separate machine states.

---

## Testing & Validation

The PLC program was functionally validated for:

- Machine startup and Ready state
- Material A filling
- Material B filling
- Timed mixing
- Drain cycle
- Batch completion and counting
- Emergency-stop response
- Mixer-overload response
- Auto/Manual mode conflict
- Level conflict
- Fill timeout
- Manual Valve A operation
- Manual Valve B operation
- Manual Mixer operation
- Manual Drain operation

**All screenshot-supported documented validation categories passed.**

Testing represents PLC/simulation validation and does not represent physical process commissioning or safety certification.

---

## Engineering Lessons

This project demonstrates several important controls-engineering concepts:

- State-based sequential process control
- Separating machine permission from operator run demand
- Explicit retained process states
- Sensor-based state transitions
- Independent timeout supervision
- Controlled process recovery
- Automatic and manual control integration
- Hold-to-run maintenance controls
- Manual process interlocks
- Single-output-coil architecture
- Process-level consistency diagnostics
- Batch completion events
- Production counting
- Deliberate use of PLC scan order
- Structured troubleshooting through internal state bits

---

## Version 1 → Version 2 Development

The original Version 1 project established the foundational batch mixing sequence. Version 2 develops that foundation into a more structured industrial batch process with clearly defined operating states, equipment permissives, sequence control, monitoring, and fault handling.

- 📄 [View Original Version 1 Project](./Version-1/Project_07_Version_1.pdf)

Version 2 expands the original application with Auto/Manual operating modes, System Ready and permissive logic, a structured Fill A → Fill B → Mix → Drain sequence, timed process stages, batch completion and counting, timeout supervision, master-fault handling, controlled recovery, and manual hold-to-run maintenance controls.

---

## Industrial Relevance

Batch mixing systems are widely used in:

- Chemical processing
- Food and beverage production
- Pharmaceutical manufacturing
- Water treatment
- Paint and coatings
- Consumer-product manufacturing

This project demonstrates the PLC architecture required to coordinate filling, mixing, draining, process supervision, fault handling, manual maintenance operation, batch counting, and operator indication.

A physical industrial implementation could additionally include analog level transmitters, flow measurement, valve-position feedback, mixer-speed feedback, recipes, process instrumentation, safety-rated hardware, HMI/SCADA integration, data logging, and formal commissioning.
