<h1 align="center">💻 Project Reports 💻</h1>
<h1 align="center">Project 3 ⚙️ Stepper Motor Control</h1>

## Introduction

**Idea:** A controller that drives a 4-coil stepper motor from the Basys3 FPGA. A **finite state machine (FSM)** produces the coil sequence, one step per clock pulse. Two inputs control it: **En** (enable) starts or stops the motor, and **Dir** sets the direction (clockwise or counter-clockwise). The FPGA output goes through Pmod port JA to a **ULN2003 driver**, which supplies the current the motor coils need.

**Objectives** (from the handout):

1. Design a controller for a stepper motor with direction control.
2. Apply the finite state machine (FSM) technique to generate the step pulses.
3. Interface the FPGA with an external circuit through the Pmod connectors.
4. Understand the basic operation of a stepper motor and its drive sequences.


## Design

**Block Diagram**

Clk (100 MHz) → `clk_div` (500 Hz) → `stepper_motor_FSM` → q[3:0] → Pmod JA → ULN2003 driver → Stepper motor
(En, Dir and Reset also connect to the FSM)

**Drive mode: wave drive.** Only one coil is energized at a time, and the four coils switch on one by one. It gives low torque but is simple, so it's good for learning.

**States**

| State | q[3:0] | Meaning |
|---|---|---|
| S0 | 0000 | Idle, all coils off (en = 0) |
| S1 | 0001 | Coil 1 on |
| S2 | 0010 | Coil 2 on |
| S3 | 0100 | Coil 3 on |
| S4 | 1000 | Coil 4 on |

**State Table**

| Present state | Next state, CW (en = 1, dir = 1) | Next state, CCW (en = 1, dir = 0) | en = 0 |
|---|---|---|---|
| S0 | S1 | S4 | S0 |
| S1 | S2 | S4 | S0 |
| S2 | S3 | S1 | S0 |
| S3 | S4 | S2 | S0 |
| S4 | S1 | S3 | S0 |

**Components**

| Module | Function |
|---|---|
| `stepper_motor_FSM` | FSM that moves one state per clock edge, CW or CCW depending on `dir`, and returns to S0 when `en = 0`. Output logic uses `with…select`. |
| `clk_div` | Reused from Project 2. Divides 100 MHz down to a 500 Hz step clock (`clk_out_freq => 500`). |
| `Stepper_Controller` | Top-level structural design that connects `clk_div` and the FSM. |
| ULN2003 (external) | Driver IC that amplifies the FPGA's small output current to drive the motor coils from +5 V. |

**Key design points**

- The FSM steps on every rising edge of the 500 Hz clock, so the motor takes **500 steps per second**.
- Motor wiring order: q(0) = D, q(1) = C, q(2) = B, q(3) = A (q = ABCD).
- The motor's Vcc and GND come from the +5 V bench power supply, not the FPGA.
