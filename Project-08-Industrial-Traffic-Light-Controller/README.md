# Project 08 — Industrial Traffic Light Controller

## Project Overview

This project develops an industrial-style traffic intersection controller using an Allen-Bradley PLC and a structured integer-based state-machine architecture.

The automatic sequence is controlled primarily by `N7:0 Traffic_State`. EQU instructions identify the active state, TON instructions provide sequence timing, and MOV instructions transition the controller between states.

The system supports automatic daytime traffic sequencing, startup All Red, North-South and East-West Green/Yellow phases, All-Red clearance intervals, pedestrian request capture and service, Night Mode flashing, Manual Mode, Emergency All Red, fault handling, controlled recovery, and traffic-signal diagnostics.

Testing was performed in a PLC/simulation environment.

---

## Project Files

- 📄 [Full Engineering Report](./Project_08_Version_2_Industrial_Traffic_Light_Controller.pdf)
- ⚙️ [RSLogix 500 PLC Program](./PV8.RSS)

> The `.RSS` file contains the native RSLogix 500 ladder-logic project. RSLogix 500 or compatible Rockwell Automation software is required to open the PLC program.

---

## Key Features

- Integer-based automatic state machine
- Central `N7:0 Traffic_State` sequence register
- Automatic, Manual, and Night operating modes
- Startup All-Red sequence
- North-South and East-West traffic phases
- All-Red clearance intervals
- Stored pedestrian request
- Pedestrian Walk and clearance sequence
- Night Mode flashing
- Separate `N7:1` manual-state register
- Emergency All-Red response
- Auto/Manual mode-conflict detection
- Latched master fault
- Controlled fault recovery
- Invalid automatic-state detection
- Traffic-signal conflict diagnostics
- Single final OTE for each physical traffic signal

---

## Control Architecture

The PLC program follows a layered control architecture:

`Inputs → Permissives / Mode Logic → State or Mode Logic → Internal Commands → Output Arbitration → Physical Signals → Diagnostics → Fault Response`

Automatic sequencing is handled by `N7:0`.

Manual operation uses the separate `N7:1 Manual_Traffic_State` register.

Night Mode uses dedicated flashing commands.

The different command layers converge only at the final physical-output rungs so that every traffic lamp has one final OTE.

---

## Automatic State Machine

| State | Function | Duration |
|---|---|---:|
| 0 | Startup All Red | 3 s |
| 10 | NS Green / EW Red | 10 s |
| 20 | NS Yellow / EW Red | 3 s |
| 30 | All-Red Clearance / Pedestrian Decision | 3 s |
| 40 | NS Red / EW Green | 10 s |
| 50 | NS Red / EW Yellow | 3 s |
| 60 | All-Red Clearance | 3 s |
| 70 | Pedestrian Walk | 7 s |
| 80 | Pedestrian Clearance | 5 s |

The normal vehicle sequence is:

`State 0 → 10 → 20 → 30 → 40 → 50 → 60 → 10`

If a pedestrian request is pending at State 30:

`State 30 → 70 → 80 → 40`

---

## Pedestrian Control

A momentary pedestrian request from `I:1/7` is stored in:

`B3:1/2 Ped_Request_Latched`

The request does not immediately interrupt an active traffic movement.

Instead, the PLC waits until the sequence reaches the State 30 All-Red clearance point.

After the clearance timer completes:

- No pedestrian request → State 40
- Pedestrian request pending → State 70

State 70 provides the 7-second Walk interval.

State 80 provides the 5-second pedestrian clearance interval using the PLC clock bit `S:4/8` for flashing behavior.

After pedestrian clearance, the controller transitions to State 40.

---

## Night Mode

Night Mode operates outside the main `N7:0` daytime state machine.

When Night Mode is selected under valid conditions:

- Normal automatic decoding is disabled
- North-South Yellow flashes
- East-West Red flashes
- Both flashing commands use `S:4/8`

Keeping Night Mode outside the main automatic state machine prevents unnecessary states from being added to `N7:0`.

---

## Manual Mode

Manual operation uses:

`N7:1 Manual_Traffic_State`

instead of modifying the automatic `N7:0 Traffic_State`.

Implemented manual states include:

| N7:1 | Manual Condition |
|---:|---|
| 0 | All Red |
| 10 | NS Green / EW Red |
| 20 | NS Yellow / EW Red |
| 40 | NS Red / EW Green |
| 50 | NS Red / EW Yellow |

Unsupported manual-state values fall back to **All Red**.

This protects the integrity of the automatic state machine while providing controlled manual traffic selection.

---

## Emergency & Fault Response

Emergency All Red and qualifying faults establish the common All-Red response.

The controller uses:

`B3:0/4 Fault_Active`

as the latched master fault.

The system does not automatically restart after a fault is cleared.

Recovery requires:

`Correct Fault → Reset → State 0 → New Start Command`

This ensures the traffic sequence returns through the startup All-Red state before another Green movement is permitted.

---

## Diagnostics

The diagnostic layer monitors the controller independently of the normal command-generation logic.

Implemented diagnostics include:

- Invalid `N7:0` automatic state
- NS Green + NS Yellow conflict
- EW Green + EW Yellow conflict
- Simultaneous opposing Green signals

Individual diagnostic bits feed:

`B3:5/4 Diagnostic_Fault`

which then participates in the master fault architecture:

`Abnormal Condition → Individual Diagnostic → Diagnostic_Fault → Fault_Active → All Red`

Several diagnostics monitor the actual physical output bits rather than only upstream internal commands.

---

## Physical Inputs

| Address | Function |
|---|---|
| I:1/0 | Start PB |
| I:1/1 | Stop PB |
| I:1/2 | Reset PB |
| I:1/3 | Auto Mode |
| I:1/4 | Manual Mode |
| I:1/5 | EStop OK |
| I:1/7 | Pedestrian Request |
| I:1/8 | Emergency All Red |
| I:1/9 | Night Mode |

---

## Physical Outputs

| Address | Function |
|---|---|
| O:2/0 | North-South Red |
| O:2/1 | North-South Yellow |
| O:2/2 | North-South Green |
| O:2/3 | East-West Red |
| O:2/4 | East-West Yellow |
| O:2/5 | East-West Green |

---

## Testing & Validation

The final controller was validated for:

- Startup and Run Control
- Automatic Traffic Sequence
- Pedestrian Sequence
- Manual Mode
- Night Mode
- Emergency / Fault Response
- Mode Transition and Conflict Handling
- Diagnostic System
- Integrated Final Acceptance

**Final documented status: PASS**

Testing was performed in the PLC/simulation environment and does not represent field commissioning of a public traffic-control installation.

---

## Engineering Lessons

This project demonstrates several transferable industrial controls concepts:

- Integer-based state-machine programming
- Separating state identification, timing, and transitions
- Operating-mode arbitration
- Internal command layers
- Output arbitration
- Single physical-output ownership
- Stored operator requests
- Safe sequence transitions
- Fault latching
- Controlled restart
- Invalid-state detection
- Output-conflict monitoring
- Fail-safe fallback behavior
- Structured PLC troubleshooting

---

## Industrial Relevance

Although implemented as a traffic-controller project, the architecture is directly transferable to industrial automation systems involving:

- Sequential machine control
- Packaging equipment
- Material-handling systems
- Batch processes
- Automated production cells
- Multi-mode machine operation
- Stored operator requests
- Equipment interlocks
- Diagnostic monitoring
- Controlled fault recovery

The primary engineering value of this project is its layered structure and deterministic state-machine architecture rather than the traffic application itself.
