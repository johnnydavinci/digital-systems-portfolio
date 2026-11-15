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

## Code

**Stepper_Controller.vhd**
````vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity Stepper_Controller is
    Port ( Clk   : in  STD_LOGIC;
           Reset : in  STD_LOGIC;
           En    : in  STD_LOGIC;
           Dir   : in  STD_LOGIC;
           q     : out STD_LOGIC_VECTOR (3 downto 0));
end Stepper_Controller;

architecture Structural of Stepper_Controller is
    -- Clock divider component declaration
    component clk_div
        Generic ( clk_in_freq  : integer;
                  clk_out_freq : integer);
        Port ( reset   : in  STD_LOGIC;
               clk_in  : in  STD_LOGIC;
               clk_out : out STD_LOGIC);
    end component;

    -- FSM component declaration
    component stepper_motor_FSM
        Port ( clk : in  STD_LOGIC;
               dir : in  STD_LOGIC;
               en  : in  STD_LOGIC;
               q   : out STD_LOGIC_VECTOR (3 downto 0));
    end component;

    -- Internal signal to connect slow clock to the FSM
    signal slow_clk : STD_LOGIC;

begin
    -- Instantiate clock divider and set output frequency to 500 Hz
    clk_divider_inst : clk_div
        generic map ( clk_in_freq  => 100000000,
                      clk_out_freq => 500)
        port map ( reset   => Reset,
                   clk_in  => Clk,
                   clk_out => slow_clk);

    -- Instantiate FSM and connect the slow_clk generated above
    fsm_inst : stepper_motor_FSM
        port map ( clk => slow_clk,
                   dir => Dir,
                   en  => En,
                   q   => q);
end Structural;
````

*Top-level structural design.* It connects the two modules:
- `clk_divider_inst` divides the 100 MHz board clock down to a 500 Hz step clock, set with `generic map (clk_out_freq => 500)`.
- `fsm_inst` uses that slow clock (`slow_clk`), so the motor takes one step every 2 ms (500 steps per second).
- `slow_clk` is an internal signal (a "wire") that joins the divider's output to the FSM's clock input.

**clk_div.vhd**
````vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity clk_div is
    Generic ( clk_in_freq  : integer := 100000000;
              clk_out_freq : integer := 500);
    Port ( reset   : in  STD_LOGIC;
           clk_in  : in  STD_LOGIC;
           clk_out : out STD_LOGIC);
end clk_div;

architecture Behavioral of clk_div is
    -- Calculate the maximum count value required to achieve the desired output frequency
    constant MAX_COUNT : integer := clk_in_freq / (2 * clk_out_freq);
    signal count       : integer range 0 to MAX_COUNT := 0;
    signal temp_clk    : STD_LOGIC := '0';
begin

    process(clk_in, reset)
    begin
        if reset = '1' then
            count <= 0;
            temp_clk <= '0';
        elsif rising_edge(clk_in) then
            if count = (MAX_COUNT - 1) then
                temp_clk <= not temp_clk;
                count <= 0;
            else
                count <= count + 1;
            end if;
        end if;
    end process;

    clk_out <= temp_clk;

end Behavioral;
````

*Frequency divider (reused from Project 2).*
- `MAX_COUNT = clk_in_freq / (2 × clk_out_freq)` = 100,000,000 / 1,000 = **100,000**.
- The counter counts input clock cycles. Every 100,000 cycles, `temp_clk` flips (toggles). Two flips make one full period, so the output is 100 MHz / 200,000 = **500 Hz**.
- **Generics** let the same code make any output frequency without editing the logic.
- **Reset** is asynchronous: it clears the counter and the output immediately.

**stepper_motor_FSM.vhd**
````vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;

ENTITY stepper_motor_FSM IS
PORT (clk, dir, en : IN STD_LOGIC;
      q            : OUT STD_LOGIC_VECTOR (3 downto 0));
END stepper_motor_FSM;

ARCHITECTURE arch OF stepper_motor_FSM IS
TYPE state_type IS (S0, S1, S2, S3, S4);
SIGNAL state: state_type := S0;

BEGIN
-- process for next state logic and state register logic
PROCESS (clk)
BEGIN
    IF clk'EVENT AND clk = '1' THEN
        IF en = '1' AND dir = '1' THEN
            -- Clockwise Steps
            CASE state IS
                WHEN S0 => state <= S1;
                WHEN S1 => state <= S2;
                WHEN S2 => state <= S3;
                WHEN S3 => state <= S4;
                WHEN S4 => state <= S1;
                WHEN OTHERS => state <= S0;
            END CASE;

        ELSIF en = '1' AND dir = '0' THEN
            -- Counter-clockwise Steps
            CASE state IS
                WHEN S0 => state <= S4;
                WHEN S1 => state <= S4;
                WHEN S2 => state <= S1;
                WHEN S3 => state <= S2;
                WHEN S4 => state <= S3;
                WHEN OTHERS => state <= S0;
            END CASE;

        ELSIF en = '0' THEN
            state <= S0;
        END IF;
    END IF;
END PROCESS;

--output logic using with..select (since it is concurrent statement, no process required)
WITH state SELECT
    q <= "0001" WHEN S1,
         "0010" WHEN S2,
         "0100" WHEN S3,
         "1000" WHEN S4,
         "0000" WHEN S0,
         "0000" WHEN OTHERS;

END arch;
````

*Finite state machine that generates the wave-drive sequence.*
- **State type:** `TYPE state_type IS (S0, S1, S2, S3, S4)` creates five named states. Vivado chooses the binary encoding automatically.
- **State register + next-state logic (process):** On each rising clock edge, the FSM moves to the next state:
  - `en = 1, dir = 1` → clockwise: S1 → S2 → S3 → S4 → S1 …
  - `en = 1, dir = 0` → counter-clockwise: S4 → S3 → S2 → S1 → S4 …
  - `en = 0` → back to S0 (all coils off, motor stops).
- **Output logic (`with…select`):** A concurrent statement that turns each state into the coil pattern on `q`. Only one bit is `1` at a time (wave drive). The output depends only on the state, so this is a **Moore machine**.

**Constraints (stepper_m.xdc)**
````tcl
## Clock signal (100 MHz)
set_property PACKAGE_PIN W5 [get_ports Clk]
set_property IOSTANDARD LVCMOS33 [get_ports Clk]
create_clock -add -name sys_clk_pin -period 10.00 -waveform {0 5} [get_ports Clk]

## Switches
set_property PACKAGE_PIN V17 [get_ports En]
set_property IOSTANDARD LVCMOS33 [get_ports En]
set_property PACKAGE_PIN V16 [get_ports Dir]
set_property IOSTANDARD LVCMOS33 [get_ports Dir]
set_property PACKAGE_PIN W16 [get_ports Reset]
set_property IOSTANDARD LVCMOS33 [get_ports Reset]

## Pmod Header JA (to ULN2003 driver)
set_property PACKAGE_PIN J1 [get_ports {q[0]}]
set_property IOSTANDARD LVCMOS33 [get_ports {q[0]}]
set_property PACKAGE_PIN L2 [get_ports {q[1]}]
set_property IOSTANDARD LVCMOS33 [get_ports {q[1]}]
set_property PACKAGE_PIN J2 [get_ports {q[2]}]
set_property IOSTANDARD LVCMOS33 [get_ports {q[2]}]
set_property PACKAGE_PIN G2 [get_ports {q[3]}]
set_property IOSTANDARD LVCMOS33 [get_ports {q[3]}]

## Configuration options
set_property CONFIG_VOLTAGE 3.3 [current_design]
set_property CFGBVS VCCO [current_design]
````

*Maps every port to a Basys3 pin:* the clock to W5 (100 MHz), `En` to SW0 (V17), `Dir` to SW1 (V16), `Reset` to SW2 (W16), and `q[3:0]` to Pmod JA pins JA1–JA4 (J1, L2, J2, G2), which connect to the ULN2003 driver inputs. All pins use LVCMOS33 (3.3 V).

## Simulation

**stepper_motor_FSM_tb.vhd**
````vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;

ENTITY stepper_motor_FSM_tb IS
END stepper_motor_FSM_tb;

ARCHITECTURE behavior OF stepper_motor_FSM_tb IS
    COMPONENT stepper_motor_FSM
    PORT(
         clk : IN  std_logic;
         dir : IN  std_logic;
         en  : IN  std_logic;
         q   : OUT std_logic_vector(3 downto 0)
        );
    END COMPONENT;

    signal clk : std_logic := '0';
    signal dir : std_logic := '0';
    signal en  : std_logic := '0';
    signal q   : std_logic_vector(3 downto 0);

    constant clk_period : time := 10 us;
BEGIN
    uut: stepper_motor_FSM PORT MAP (
          clk => clk,
          dir => dir,
          en  => en,
          q   => q
        );

    clk_process : process
    BEGIN
        clk <= '0';
        wait for clk_period/2;
        clk <= '1';
        wait for clk_period/2;
    END process;

    stim_proc: process
    BEGIN
        -- Motor disabled at start (S0)
        wait for 2 * clk_period;

        -- Clockwise rotation for 4 clock cycles
        en  <= '1';
        dir <= '1';
        wait for 4 * clk_period;

        -- Counter-clockwise rotation for 4 clock cycles
        dir <= '0';
        wait for 4 * clk_period;

        -- Disable: motor stops and returns to S0
        en <= '0';
        wait for 2 * clk_period;

        wait;
    END process;
END behavior;
````

*The testbench generates a 10 µs clock (as the handout asks) and tests three cases:*
1. **Clockwise** (`en = 1, dir = 1`) for 4 clock cycles.
2. **Counter-clockwise** (`en = 1, dir = 0`) for 4 clock cycles.
3. **Disable** (`en = 0`): the FSM returns to S0 and all coils turn off.

<!-- ADD FSM WAVEFORM IMAGE HERE -->

| Clock edge | en | dir | state | q |
|---|---|---|---|---|
| Before enable | 0 | 0 | S0 | 0000 |
| 1 | 1 | 1 | S1 | 0001 |
| 2 | 1 | 1 | S2 | 0010 |
| 3 | 1 | 1 | S3 | 0100 |
| 4 | 1 | 1 | S4 | 1000 |
| 5 | 1 | 0 | S3 | 0100 |
| 6 | 1 | 0 | S2 | 0010 |
| 7 | 1 | 0 | S1 | 0001 |
| 8 | 1 | 0 | S4 | 1000 |
| 9 | 0 | 0 | S0 | 0000 |

*Result:* With `dir = 1` the single `1` in `q` shifts left (0001 → 0010 → 0100 → 1000), turning the motor clockwise. With `dir = 0` it shifts right, turning it counter-clockwise. When `en = 0` the FSM goes to S0 and `q = 0000`. All results match the state table.

**clk_div_tb.vhd**
````vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;

ENTITY Clk_div_tb IS
END Clk_div_tb;

ARCHITECTURE behavior OF Clk_div_tb IS
    COMPONENT clk_div
    GENERIC ( clk_in_freq  : integer;
              clk_out_freq : integer);
    PORT(
         reset   : IN  std_logic;
         clk_in  : IN  std_logic;
         clk_out : OUT std_logic
        );
    END COMPONENT;

    signal reset   : std_logic := '0';
    signal clk_in  : std_logic := '0';
    signal clk_out : std_logic;

    -- 100 MHz master clock period
    constant clk_in_period : time := 10 ns;
BEGIN
    uut: clk_div
    GENERIC MAP (
          clk_in_freq  => 100000000,
          clk_out_freq => 5000000 -- Artificially high for faster simulation visibility
        )
    PORT MAP (
          reset   => reset,
          clk_in  => clk_in,
          clk_out => clk_out
        );

    clk_in_process : process
    begin
        clk_in <= '0';
        wait for clk_in_period/2;
        clk_in <= '1';
        wait for clk_in_period/2;
    end process;

    stim_proc: process
    begin
        reset <= '1';
        wait for 100 ns;
        reset <= '0';
        wait;
    end process;
END behavior;
````

*Checks the clock divider.* The output is set to 5 MHz only for simulation, so `MAX_COUNT = 10` and `clk_out` toggles every 10 input clocks (one full period = 200 ns). At the real 500 Hz, one period is 2 ms, far too long to see clearly in a simulation.

<!-- ADD CLK_DIV WAVEFORM IMAGE HERE -->

*Result:* `clk_out` stays at 0 during reset, then toggles every 100 ns, giving a 200 ns period (5 MHz), which matches the calculation.

## Hardware

**Elaborated Design**\
<!-- ADD ELABORATED DESIGN IMAGE HERE -->

*Shows the structure from the VHDL:* `clk_divider_inst` takes `Clk` and `Reset` and outputs `slow_clk`, which feeds the clock input of `fsm_inst`. `En` and `Dir` go straight to the FSM, and the FSM drives `q[3:0]`.

**Synthesized Design**\
<!-- ADD SYNTHESIZED DESIGN IMAGE HERE -->

*Shows the design mapped to FPGA resources:* input buffers (IBUF) for the switches, a global clock buffer (BUFG) for the 100 MHz clock, the clock divider and FSM in the middle (flip-flops and LUTs), and 4 output buffers (OBUF) for `q[3:0]` going to Pmod JA.

**Implemented Design**\
<!-- ADD IMPLEMENTED DESIGN IMAGE HERE -->

*Shows where the logic was placed on the Artix-7 chip.* Only a small area is used, close to the I/O pins for the switches and Pmod JA. The rest of the chip is empty.

**Utilization report**\
<!-- ADD UTILIZATION REPORT IMAGE HERE -->

| Resource | Used | Available |
|---|---|---|
| Slice LUTs | ? | 20,800 |
| Slice Registers (flip-flops) | ? | 41,600 |
| Bonded IOB | 8 | 106 |
| BUFGCTRL (clock buffer) | ? | 32 |

Most of the registers come from the clock divider, whose counter must reach 100,000 (17 bits), plus `temp_clk`. The FSM needs only a few flip-flops to store its 5 states. The 8 IOBs are 1 clock + 3 switches (En, Dir, Reset) + 4 outputs (`q`).

**Basys3 behaviour**

After programming, the FPGA was connected to the ULN2003 driver through Pmod JA, with the motor powered from the +5 V bench supply. With SW0 (En) on and SW1 (Dir) on, the motor turned clockwise; with SW1 off it turned counter-clockwise. With SW0 off the motor stopped. The four LEDs on the driver board lit one at a time in the same order as `q`.

## Reflection

**Challenges and resolutions**
- **Testbench timing:** The first testbench waited 20 ms per direction. With a 10 µs clock, that is 2,000 steps, so the waveform was too crowded to read. Using `4 * clk_period` showed exactly 4 steps in each direction.
- **Counter-clockwise sequence:** The handout's state table showed S1 → S0 for counter-clockwise, but the state diagram shows S1 → S4. Going to S0 would stop the motor, so S1 → S4 was used.
- **Simulating the clock divider:** At 500 Hz one period is 2 ms (200,000 input clocks), too long to view. The testbench used `clk_out_freq => 5000000` so the toggling was visible.
- **Wiring the motor:** The coil order had to match q (q(0) = D, q(1) = C, q(2) = B, q(3) = A). The motor was powered from the +5 V supply, not the FPGA, because the coils need more current than the FPGA pins can give.

**Learning linked to theory**
- **Finite state machines:** The design is a Moore machine: the output `q` depends only on the current state. The process holds the state register and next-state logic, and `with…select` is the output logic.
- **Wave drive:** Only one coil is on at a time, so the 1 in `q` shifts one position each step. The direction of the shift decides the direction of rotation.
- **Clock division and generics:** `clk_div` from Project 2 was reused without changes; only the generic `clk_out_freq => 500` was set. The step frequency directly controls the motor speed.
- **Interfacing with external hardware:** The FPGA's 3.3 V outputs can't drive a motor directly. The ULN2003 driver amplifies the current, which shows why a driver stage is needed between logic and power devices.
