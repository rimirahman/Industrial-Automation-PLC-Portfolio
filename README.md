# Industrial Automation & PLC Controls Engineering Portfolio

A hands-on controls engineering portfolio demonstrating the progressive development of Allen-Bradley PLC applications from foundational control logic to structured industrial automation systems.

The portfolio contains **10 PLC projects**, each developed first as a foundational Version 1 application and then upgraded into a Version 2 industrial implementation with increased emphasis on operating modes, permissives, interlocks, diagnostics, fault handling, maintainability, verification, and commissioning.

## Technical Focus

- Allen-Bradley PLC programming
- RSLogix 500 / MicroLogix control architecture
- Ladder Logic development
- Auto/Manual operating modes
- Machine permissives and interlocks
- Sequential machine and process control
- Timer and counter applications
- Fault detection and diagnostics
- Equipment feedback verification
- Controlled shutdown and recovery
- Production counting and monitoring
- PLC testing and commissioning

## PLC Projects

| # | Project | Main Engineering Focus |
|---|---|---|
| 01 | [Industrial Motor Control System](./Project-01-Industrial-Motor-Control-System) | Auto/Manual control, permissives, startup delay, feedback verification, fault handling, runtime monitoring |
| 02 | [Industrial Batch Conveyor Counter System](./Project-02-Industrial-Batch-Conveyor-Counter-System) | Configurable batch counting, conveyor sequencing, jam detection, batch completion |
| 03 | [Water Tank Pump Control System](./Project-03-Water-Tank-Pump-Control-System) | Level-based pump control, Auto/Manual operation, dry-run monitoring, fault handling |
| 04 | [Industrial Temperature Control System](./Project-04-Industrial-Temperature-Control-System) | Heating/cooling control, process-value monitoring, sensor diagnostics, overtemperature protection |
| 05 | [Industrial Bottle Filling and Capping System](./Project-05-Industrial-Bottle-Filling-and-Capping-System) | Conveyor indexing, filling and capping sequencing, production counting, machine-state control |
| 06 | [Industrial Conveyor Sorting System](./Project-06-Industrial-Conveyor-Sorting-System) | Product tracking, retained classification, automatic reject handling, reject confirmation, jam diagnostics |
| 07 | [Industrial Batch Mixing System](./Project-07-Industrial-Batch-Mixing-System) | Fill → Mix → Drain sequencing, process-state control, timeout supervision, manual maintenance controls |
| 08 | [Industrial Traffic Light Controller](./Project-08-Industrial-Traffic-Light-Controller) | Integer state machine, Auto/Manual/Night modes, pedestrian sequence, emergency operation, diagnostics |
| 09 | [Three-Floor Elevator Controller](./Project-09-Three-Floor-Elevator-Controller) | Stored floor requests, position tracking, motion permissives, door sequencing, travel supervision |
| 10 | [Industrial Packaging and Sorting Station](./Project-10-Industrial-Packaging-and-Sorting-Station) | Integrated machine architecture, reject verification, diagnostics, batch control, quality monitoring, production statistics |

## Engineering Development Progression

Each project demonstrates a deliberate **Version 1 → Version 2 engineering progression**.

**Version 1** focuses on establishing the fundamental PLC control sequence and core machine functionality.

**Version 2** develops the same application toward a more industrial control architecture by incorporating appropriate features such as:

- Auto/Manual operating modes
- Run-request and System Ready logic
- Centralized permissives
- Equipment and process interlocks
- State-based sequencing
- Feedback verification
- Sensor and equipment diagnostics
- Timeout monitoring
- Master-fault handling
- Controlled recovery
- Production monitoring
- Maintenance-oriented manual controls

The Version 1 engineering documentation is retained within each project so that the development progression can be reviewed directly.

## Project Documentation

Each project folder contains:

- **README.md** — project overview, architecture, control strategy, I/O, sequence, diagnostics, testing, and engineering lessons
- **Version 2 Engineering PDF** — detailed project documentation and ladder-logic review
- **PLC Program (.RSS)** — RSLogix 500 project file where included
- **Version-1/** — original foundational project documentation for comparison with the industrial upgrade

## Engineering Skills Demonstrated

Across the portfolio, the projects demonstrate practical experience with:

- Translating process requirements into PLC control logic
- Separating operator requests, machine states, commands, permissives, and physical outputs
- Designing sequential control systems
- Developing equipment interlocks and operating permissives
- Implementing fault detection and diagnostic logic
- Designing controlled machine recovery behavior
- Tracking process and production information
- Testing normal, abnormal, and fault conditions
- Reviewing PLC behavior against defined commissioning scenarios
- Documenting control-system architecture and engineering rationale

## Tools & Technologies

- Allen-Bradley / Rockwell Automation PLC concepts
- RSLogix 500
- MicroLogix 1100
- Ladder Logic
- Digital and analog I/O concepts
- Timers, counters, integer registers, internal bits, comparison and data instructions
- PLC simulation and functional testing

## Portfolio Objective

This portfolio was developed to demonstrate practical controls-engineering capability beyond basic ladder-logic programming.

The projects emphasize the engineering decisions required to make automated equipment safer, more diagnosable, maintainable, testable, and suitable for structured industrial operation.

---

**Career Focus:** Controls Engineering | Industrial Automation | PLC Programming | Electrical Controls | SCADA
