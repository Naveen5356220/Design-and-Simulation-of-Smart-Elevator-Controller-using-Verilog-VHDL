# Smart Elevator Controller — Verilog / VHDL

A hardware description language (HDL) implementation of a smart elevator control system, simulated and verified using both **Verilog** and **VHDL**. The design models a real-world multi-floor elevator using a Finite State Machine (FSM), with support for floor request scheduling, door control, direction management, and emergency handling.

---

## Overview

Modern elevator systems require precise, reliable control logic to handle simultaneous floor requests, directional movement, and safety constraints. This project implements a fully digital elevator controller from the ground up — starting at the RTL level — and validates it through comprehensive simulation testbenches.

The controller is built around a Moore-type FSM that transitions between well-defined states based on sensor inputs and user requests. The SCAN (elevator) scheduling algorithm is used to optimise the order in which floor requests are serviced, minimising travel time and unnecessary direction changes.

---

## Block Diagram

> See the interactive block diagram above for a visual overview of the input/output/control architecture.

The system is divided into three layers:

- **Inputs** — floor call buttons, cabin selection buttons, floor sensors, door sensors, emergency stop, clock, and reset
- **FSM Controller** — the central control logic implemented in Verilog/VHDL, containing all state transitions and the SCAN scheduling algorithm
- **Outputs** — motor drive signals (up/down), door actuator signals (open/close), floor display, direction LEDs, and emergency alert

---

## Features

- Multi-floor elevator control (configurable number of floors)
- FSM-based control logic with clearly defined states
- SCAN algorithm for efficient floor request scheduling
- Door open/close sequencing with timer-based hold
- Emergency stop with graceful state override
- 7-segment or LED display output for current floor
- Direction indicator (up/down LED)
- Full testbench for simulation and verification
- Dual-language implementation: **Verilog HDL** and **VHDL**

---

## FSM States

| State | Description |
|---|---|
| `IDLE` | Elevator stationary, no pending requests |
| `MOVING_UP` | Elevator travelling upward toward a requested floor |
| `MOVING_DOWN` | Elevator travelling downward toward a requested floor |
| `DOOR_OPEN` | Elevator stopped at floor, door opening and holding |
| `DOOR_CLOSE` | Door closing before the next move |
| `EMERGENCY` | Emergency stop activated; all motion halted |

State transitions are driven by floor sensor feedback, pending request queues, and emergency override signals.

---

## Input / Output Signals

### Inputs

| Signal | Width | Description |
|---|---|---|
| `clk` | 1-bit | System clock |
| `rst` | 1-bit | Synchronous reset |
| `floor_req_up[N]` | N-bit | Hall call buttons (up) at each floor |
| `floor_req_dn[N]` | N-bit | Hall call buttons (down) at each floor |
| `cabin_req[N]` | N-bit | Cabin panel floor selection buttons |
| `floor_sensor[N]` | N-bit | Sensors indicating elevator position |
| `door_sensor` | 1-bit | Door open/closed feedback |
| `emergency` | 1-bit | Emergency stop signal |

### Outputs

| Signal | Width | Description |
|---|---|---|
| `motor_up` | 1-bit | Drive motor upward |
| `motor_dn` | 1-bit | Drive motor downward |
| `door_open` | 1-bit | Open door actuator |
| `door_close` | 1-bit | Close door actuator |
| `floor_display[7:0]` | 8-bit | 7-segment encoded current floor |
| `dir_led_up` | 1-bit | Direction indicator LED (up) |
| `dir_led_dn` | 1-bit | Direction indicator LED (down) |
| `emergency_alert` | 1-bit | Buzzer / display on emergency |

---

## File Structure

```
├── verilog/
│   ├── elevator_controller.v       # Top-level Verilog module
│   ├── fsm_states.v                # FSM state definitions
│   ├── request_scheduler.v         # SCAN algorithm scheduler
│   ├── door_controller.v           # Door open/close timing logic
│   └── elevator_tb.v               # Verilog testbench
│
├── vhdl/
│   ├── elevator_controller.vhd     # Top-level VHDL entity
│   ├── fsm_states.vhd              # FSM state type definitions
│   ├── request_scheduler.vhd       # SCAN scheduler in VHDL
│   ├── door_controller.vhd         # Door controller entity
│   └── elevator_tb.vhd             # VHDL testbench
│
├── sim/
│   ├── waveform_screenshots/       # Simulation output waveforms
│   └── run_sim.do                  # ModelSim/QuestaSim run script
│
└── README.md
```

---

## Getting Started

### Prerequisites

- **ModelSim** or **QuestaSim** (recommended), or
- **Vivado Simulator** (for Xilinx FPGA targets), or
- **Icarus Verilog + GTKWave** (open-source, Verilog only)

### Running the Simulation (Verilog — Icarus Verilog)

```bash
# Compile
iverilog -o elevator_sim verilog/elevator_controller.v verilog/elevator_tb.v

# Simulate
vvp elevator_sim

# View waveforms
gtkwave dump.vcd
```

### Running the Simulation (VHDL — GHDL)

```bash
# Analyse
ghdl -a vhdl/elevator_controller.vhd vhdl/elevator_tb.vhd

# Elaborate
ghdl -e elevator_tb

# Run
ghdl -r elevator_tb --vcd=dump.vcd

# View waveforms
gtkwave dump.vcd
```

### ModelSim / QuestaSim

Open ModelSim, navigate to the `sim/` directory, and run:

```tcl
do run_sim.do
```

---

## Scheduling Algorithm

The controller uses the **SCAN algorithm** (also known as the elevator algorithm) to process floor requests:

1. The elevator continues in its current direction, servicing all requests along the way.
2. When no more requests exist in that direction, the elevator reverses direction.
3. Emergency requests override the queue immediately, halting all motion.

This approach avoids starvation of any floor and minimises total travel distance compared to a naive FIFO queue.

---

## Simulation Results

Key scenarios validated by the testbench:

- Elevator starts idle, receives simultaneous up and down requests, services them in SCAN order
- Door hold timer correctly delays departure after opening
- Emergency stop overrides mid-movement and transitions to the EMERGENCY state
- Reset signal correctly returns the system to IDLE from any state
- Direction LEDs and floor display update correctly at each floor arrival

---

## FPGA Implementation Notes

The design is synthesisable and has been validated on:

- **Xilinx Nexys A7** (Vivado toolchain)
- **Terasic DE0-CV / DE2-115** (Intel Quartus Prime)

For FPGA deployment, set the clock divider to slow the FSM transitions to a human-observable speed (e.g. 1 Hz for demo purposes). Map `floor_display` to the on-board 7-segment display and `dir_led_up`/`dir_led_dn` to onboard LEDs.

---

## Tools Used

| Tool | Purpose |
|---|---|
| Verilog HDL | RTL design (primary) |
| VHDL | RTL design (alternate) |
| ModelSim / GHDL | Simulation |
| GTKWave | Waveform analysis |
| Xilinx Vivado / Intel Quartus | Synthesis & FPGA implementation |

---

## License

This project is open-source and available under the [MIT License](LICENSE).

---

## Author

**Naveen** — [GitHub Profile](https://github.com/Naveen5356220)

Contributions, issues, and pull requests are welcome.
