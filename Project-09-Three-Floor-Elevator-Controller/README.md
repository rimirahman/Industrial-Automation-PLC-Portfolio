# Project 09 — Three-Floor Elevator Controller

## Project Overview

This project develops a structured PLC-based three-floor elevator controller using an Allen-Bradley MicroLogix PLC and RSLogix Micro Starter Lite.

The controller stores momentary floor requests, tracks the elevator's last confirmed floor, determines the required direction of travel, controls UP and DOWN movement, coordinates an automatic door sequence, supports Manual Mode, supervises travel time, and detects abnormal operating conditions.

The control architecture deliberately separates operator requests, stored states, direction decisions, motion permission, final movement commands, physical outputs, door sequencing, and diagnostics.

This project was developed and functionally tested in a PLC/emulator environment. It is not a certified passenger-elevator control or safety system.

---

## Project Files

- 📄 [Full Engineering Report](./Project_09_Version_2_Three_Floor_Elevator_Controller.pdf)
- ⚙️ [RSLogix 500 PLC Program](./PV9.RSS)

> The `.RSS` file contains the native RSLogix 500 ladder-logic project. RSLogix 500 or compatible Rockwell Automation software is required to open the PLC program.

---

## Key Features

- Three-floor automatic elevator control
- Stored floor requests
- Last-confirmed-floor tracking using `N7:0`
- Automatic UP/DOWN direction determination
- Separate Automatic and Manual direction commands
- Common Motion Permissive
- Door-closed movement interlock
- Door-cycle movement interlock
- Final UP/DOWN mutual interlocking
- Automatic arrival detection
- Automatic door opening, dwell, and closing sequence
- Door Closed feedback confirmation
- Direction-conflict diagnostics
- Invalid floor-sensor detection
- Travel-time supervision
- Latched Travel Timeout fault
- Master fault shutdown
- Fault indication
- Controlled Reset
- Separate Start required after fault recovery

---

## Control Architecture

The elevator movement path follows:

`Floor Request → Stored Request → Direction Decision → Motion Permissive → Final Direction Command → Motor Output`

Position tracking follows:

`Floor Sensor → N7:0 Current_Floor`

Arrival and door operation follow:

`Destination Sensor + Stored Request → Arrival → Door Cycle → Open → Dwell → Close → Door Closed Confirmation`

Fault handling follows:

`Diagnostic Condition → Fault Active → Run Request Removed → Motion Disabled → Controlled Reset → Separate Restart`

This layered architecture provides useful intermediate troubleshooting points instead of connecting floor requests directly to motor outputs.

---

## PLC Platform

| Item | Implementation |
|---|---|
| PLC | Allen-Bradley MicroLogix |
| Programming Software | RSLogix Micro Starter Lite |
| Programming Language | Ladder Logic |
| Current Floor Register | N7:0 |
| Door Timer | T4:0 |
| Door Timer Setting | TB 1.0 / PRE 20 |
| Travel Supervision Timer | T4:1 |
| Travel Timer Setting | TB 1.0 / PRE 60 |

---

## Physical Inputs

| Address | Function |
|---|---|
| I:1/0 | Start PB |
| I:1/1 | Stop PB |
| I:1/2 | Reset PB |
| I:1/3 | Floor 1 Request PB |
| I:1/4 | Floor 2 Request PB |
| I:1/5 | Floor 3 Request PB |
| I:1/6 | Floor 1 Sensor |
| I:1/7 | Floor 2 Sensor |
| I:1/8 | Floor 3 Sensor |
| I:1/9 | Door Closed |
| I:1/10 | Manual Mode |
| I:1/11 | Manual UP PB |
| I:1/12 | Manual DOWN PB |
| I:1/13 | Safety OK |

---

## Physical Outputs

| Address | Function |
|---|---|
| O:2/0 | Motor UP |
| O:2/1 | Motor DOWN |
| O:2/2 | Door Open |
| O:2/3 | Door Close |
| O:2/4 | Fault Indicator |

---

## Floor Position Tracking

The three floor sensors update:

`N7:0 Current_Floor`

with the corresponding floor number:

- Floor 1 Sensor → `N7:0 = 1`
- Floor 2 Sensor → `N7:0 = 2`
- Floor 3 Sensor → `N7:0 = 3`

When the elevator travels between floors and no floor sensor is active, `N7:0` retains the last confirmed floor.

This allows direction decisions to be made from a known reference position rather than requiring continuous position feedback.

---

## Stored Floor Requests

Momentary floor-request pushbuttons are converted into retained internal requests.

The PLC stores requests for:

- Floor 1
- Floor 2
- Floor 3

A request therefore remains active after the passenger releases the pushbutton.

The corresponding stored request is cleared after the destination is reached or during controlled fault recovery.

---

## Automatic Direction Control

Automatic direction decisions use the combination of:

`Current Floor + Stored Destination Request`

Examples include:

- Floor 1 + Floor 2 request → UP
- Floor 1 + Floor 3 request → UP
- Floor 2 + Floor 3 request → UP
- Floor 3 + Floor 2 request → DOWN
- Floor 3 + Floor 1 request → DOWN
- Floor 2 + Floor 1 request → DOWN

Automatic UP and DOWN decisions are inhibited while Manual Mode is selected.

---

## Manual Operation

Manual Mode provides separate:

- Manual UP command
- Manual DOWN command

Manual commands do not directly energize the motor outputs.

They pass through the same downstream Motion Permissive and direction-interlock architecture used by automatic operation.

This allows controlled manual movement without bypassing the fundamental motion restrictions.

---

## Motion Permissive

Movement requires:

- System Ready
- Door Closed feedback
- No active Door Cycle

The common logic can be represented as:

`System Ready + Door Closed + No Door Cycle → Motion Permissive`

Both automatic and manual movement depend on this permission.

Therefore, opening the door or activating the automatic door cycle removes permission for elevator movement.

---

## Final Direction Commands

Automatic and Manual direction requests are combined only at the final-command layer.

For UP movement:

`Auto UP OR Manual UP → Motion Permissive → Opposite Direction Interlock → Final UP Command`

For DOWN movement:

`Auto DOWN OR Manual DOWN → Motion Permissive → Opposite Direction Interlock → Final DOWN Command`

The final UP and DOWN commands are mutually interlocked.

The physical motor outputs are then driven only by their corresponding final commands.

---

## Automatic Door Sequence

When the elevator reaches a requested destination, the PLC captures the arrival before clearing the stored request.

This activates:

`Door_Cycle_Active`

The sequence then coordinates:

1. Destination arrival
2. Door-cycle activation
3. Door opening
4. Timed dwell
5. Door closing
6. Door Closed feedback confirmation
7. Door-cycle completion

Timer completion alone does not restore movement permission.

The controller also requires actual `Door_Closed` feedback before the door cycle is considered complete.

---

## Travel Supervision

`T4:1` supervises elevator travel while a final UP or DOWN movement command is active.

The final commissioning setting is:

`T4:1 = Time Base 1.0 / Preset 60`

Normally, arrival at the requested destination removes the movement command before the timer expires.

If movement continues excessively without expected arrival feedback:

`T4:1/DN → Travel Timeout`

The Travel Timeout condition is latched.

This is important because the resulting master fault removes movement, which resets the non-retentive travel timer. Latching the diagnostic preserves evidence of the original timeout for troubleshooting.

---

## Diagnostics

The controller monitors several abnormal operating conditions.

### Direction Conflict

The diagnostic layer detects contradictory direction requests.

This is separate from the final UP/DOWN mutual interlock.

The interlock prevents contradictory physical movement, while the diagnostic identifies that an abnormal direction-request condition occurred.

### Invalid Floor Sensors

The PLC detects invalid simultaneous combinations of the Floor 1, Floor 2, and Floor 3 sensors.

The elevator should not be positively confirmed at multiple floors simultaneously.

### Travel Timeout

Excessive commanded movement without expected arrival feedback generates a latched Travel Timeout.

These diagnostic conditions participate in the master fault architecture.

---

## Fault Handling & Recovery

When a qualifying diagnostic condition activates the master fault:

- Run Request is removed
- System Ready drops
- Motion Permissive drops
- Final movement commands drop
- Motor movement stops
- Fault Indicator energizes

Recovery is deliberate.

The operator must:

`Correct Fault → Reset → Separate Start`

Reset also clears stored floor requests so the elevator returns to a known request state.

The controller therefore does not automatically resume movement immediately after a fault condition disappears.

---

## Testing & Validation

The final PLC program was functionally validated for:

- Startup and Ready state
- Motion Permissive
- Floor 1 → Floor 2 automatic UP travel
- Floor 2 → Floor 3 automatic UP travel
- Floor 3 → Floor 1 automatic DOWN travel
- Destination arrival
- Automatic door sequence
- Door Closed confirmation
- Manual UP operation
- Manual DOWN operation
- Door-open motion inhibition
- Invalid floor-sensor detection
- Travel Timeout
- Fault shutdown
- Fault Reset
- Stored-request recovery
- Controlled restart
- Final integrated acceptance

**Final documented integrated acceptance: PASS**

Testing was performed in the PLC/emulator environment using manually simulated floor sensors, door feedback, mode selection, and operator commands.

---

## Engineering Lessons

This project demonstrates several important controls-engineering concepts:

- Separating operator requests from physical outputs
- Retaining momentary requests
- Last-confirmed-position tracking
- Automatic direction determination
- Separate Auto and Manual command layers
- Common motion permissives
- Mutual direction interlocking
- Feedback-based motion authorization
- Arrival capture and scan-order considerations
- Automatic door sequencing
- Feedback-confirmed sequence completion
- Travel supervision
- Latched diagnostic faults
- Master-fault architecture
- Controlled fault recovery
- Deliberate restart after Reset
- Structured PLC troubleshooting

---

## Version 1 → Version 2 Development

The original Version 1 project established the foundational three-floor elevator movement and floor-selection logic. Version 2 develops that foundation into a more structured industrial-style elevator control system with stored requests, position tracking, motion authorization, door sequencing, diagnostics, and controlled recovery.

- 📄 [View Original Version 1 Project](./Version-1/Project_09_Version_1.pdf)

Version 2 expands the original application with stored floor requests, Current Floor tracking, Auto/Manual operating modes, separate automatic and manual direction-command layers, centralized Motion Permissive logic, door-cycle sequencing, travel-time supervision, direction-conflict diagnostics, master-fault handling, and controlled fault recovery.

---

## Industrial Relevance

Although this is not a certified passenger-elevator controller, the control concepts are applicable to:

- Industrial lifts
- Hoists
- Vertical material-handling systems
- Transfer systems
- Automated storage and retrieval equipment
- Positioning systems
- Multi-position machinery

The primary engineering value is the separation of:

`Request → Decision → Authorization → Final Command → Physical Output`

combined with feedback-based sequencing, motion interlocks, diagnostics, and controlled recovery.

A real passenger elevator would require certified safety systems, applicable codes and standards, redundant safety architecture, door-lock monitoring, overspeed protection, braking systems, emergency functions, and specialized elevator hardware.
