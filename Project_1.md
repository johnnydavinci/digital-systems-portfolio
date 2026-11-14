<h1 align="center">💻 Project Reports 💻
<br>
🚗 Project 1 – Car Seat Belt Warning Light<br></h1>

<img width="2752" height="1536" alt="AdobeExpressPhotos_28d75d22bb47427495b3e996df47e723_CopyEdited" src="https://github.com/user-attachments/assets/8af50113-0cbe-40cd-8e9f-631bd5e652f9" />

   
  ## Introduction
  
  **Idea**: A digital control circuit that turns on a car’s seat-belt warning light (L) based on four sensor inputs. A sensor sets its output to 1 when its condition is true; for example, the pressure sensor outputs 1 when someone is in the seat.
  
| Input | = 1 when… | Basys3 switch / pin |
|---|---|---|
| D (door_closed) | door is closed | SW0 / V17 |
| K (key_in) | key is in the ignition | SW1 / V16 |
| P (seat_pressure) | a person is in the seat | SW2 / W16 |
| S (seat_belt) | seat belt is fastened | SW3 / W17 |
| W (output) | warning light is ON | LED0 / U16 |
| STATUS[1:0] | warning type | LED2 / U19, LED1 / E19 |
  
  **Objectives**
  
  1. Design the circuit the traditional way: truth table, K-map and Boolean simplification. (Part I)
  2. Implement the same circuit on the Basys3 FPGA (Artix-7 xc7a35tcpg236-1) through the Vivado flow: design entry, simulation, synthesis,              implementation, bitstream and programming. (Part II)
  3. Challenge: extend the design so a 2-bit STATUS output tells apart a door warning and a seat-belt warning. (Part III)
  
  
  ## Design


**Simple block diagram**


Switches (D, K, P, S) → car_warnlight logic (FPGA) → LEDs (W, STATUS[1:0])


**Truth Table**


| m | D | K | P | S | W | STATUS | Meaning |
|---|---|---|---|---|---|---|---|
| 0–3 | 0 | 0 | x | x | 0 | 00 | No warning (key out) |
| 4 | 0 | 1 | 0 | 0 | 1 | 01 | Door warning |
| 5 | 0 | 1 | 0 | 1 | 1 | 01 | Door warning |
| 6 | 0 | 1 | 1 | 0 | 1 | 11 | Both warnings |
| 7 | 0 | 1 | 1 | 1 | 1 | 01 | Door warning |
| 8–11 | 1 | 0 | x | x | 0 | 00 | No warning (key out) |
| 12 | 1 | 1 | 0 | 0 | 0 | 00 | No warning |
| 13 | 1 | 1 | 0 | 1 | 0 | 00 | No warning |
| 14 | 1 | 1 | 1 | 0 | 1 | 10 | Seat-belt warning |
| 15 | 1 | 1 | 1 | 1 | 0 | 00 | No warning |

**W = Σm(4, 5, 6, 7, 14)**


**K-map**


| DK \ PS | 00 | 01 | 11 | 10 |
|---------|----|----|----|----|
| **00**  | 0  | 0  | 0  | 0  |
| **01**  | 1  | 1  | 1  | 1  |
| **11**  | 0  | 0  | 0  | 1  |
| **10**  | 0  | 0  | 0  | 0  |


**Boolean Simplification**


`L = D'K + KPS' = K(D' + PS')`

  
  ## Code
  

**VHDL**


```
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity car_warnlight is
    Port ( D      : in STD_LOGIC;  -- Door closed
           K      : in STD_LOGIC;  -- Key in
           P      : in STD_LOGIC;  -- Person on seat
           S      : in STD_LOGIC;  -- Seat belt fastened
           W      : out STD_LOGIC; -- Warning Light
           STATUS : out STD_LOGIC_VECTOR (1 downto 0)); -- Warning Status
end car_warnlight;

architecture Behavioral of car_warnlight is
    -- Internal signals for specific warning conditions
    signal door_warning : STD_LOGIC;
    signal belt_warning : STD_LOGIC;
begin
    -- The key is in and the door is open
    door_warning <= K and not D;
    
    -- The key is in, a person is seated, and the belt is not fastened
    belt_warning <= K and P and not S;

    -- The warning light W turns ON whenever either warning is active
    W <= door_warning or belt_warning;

    -- Conditional signal assignment to generate the four STATUS values
    STATUS <= "11" when (door_warning = '1' and belt_warning = '1') else
              "10" when (door_warning = '0' and belt_warning = '1') else
              "01" when (door_warning = '1' and belt_warning = '0') else
              "00";
end Behavioral;
```


**Testbench**


```
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity car_warnlight_tb is
end car_warnlight_tb;

architecture behavior of car_warnlight_tb is
    component car_warnlight
    Port ( D : in STD_LOGIC;
           K : in STD_LOGIC;
           P : in STD_LOGIC;
           S : in STD_LOGIC;
           W : out STD_LOGIC;
           STATUS : out STD_LOGIC_VECTOR (1 downto 0));
    end component;

    signal D : STD_LOGIC := '0';
    signal K : STD_LOGIC := '0';
    signal P : STD_LOGIC := '0';
    signal S : STD_LOGIC := '0';
    signal W : STD_LOGIC;
    signal STATUS : STD_LOGIC_VECTOR (1 downto 0);
begin
    uut: car_warnlight port map (
          D => D,
          K => K,
          P => P,
          S => S,
          W => W,
          STATUS => STATUS
        );

    stim_proc: process
    begin
        -- Test Case 1: No warning (Key out)
        D <= '1'; K <= '0'; P <= '0'; S <= '0';
        wait for 20 ns;
        
        -- Test Case 2: Door warning only (Key in, Door open, No person)
        D <= '0'; K <= '1'; P <= '0'; S <= '0';
        wait for 20 ns;

        -- Test Case 3: Seat-belt warning only (Key in, Door closed, Person seated, Belt off)
        D <= '1'; K <= '1'; P <= '1'; S <= '0';
        wait for 20 ns;

        -- Test Case 4: Both warnings (Key in, Door open, Person seated, Belt off)
        D <= '0'; K <= '1'; P <= '1'; S <= '0';
        wait for 20 ns;
        
        -- Test Case 5: No warning (Key in, Door closed, Person seated, Belt fastened)
        D <= '1'; K <= '1'; P <= '1'; S <= '1';
        wait for 20 ns;

        wait;
    end process;
end behavior;
```

**Contrast** 

```
tcl
## Switches (inputs)
set_property PACKAGE_PIN V17 [get_ports D]
set_property PACKAGE_PIN V16 [get_ports K]
set_property PACKAGE_PIN W16 [get_ports P]
set_property PACKAGE_PIN W17 [get_ports S]

## LEDs (outputs)
set_property PACKAGE_PIN U16 [get_ports W]
set_property PACKAGE_PIN E19 [get_ports {STATUS[0]}]
set_property PACKAGE_PIN U19 [get_ports {STATUS[1]}]

## I/O standard for all used ports
set_property IOSTANDARD LVCMOS33 [get_ports {D K P S W STATUS[*]}]

## Configuration
set_property CONFIG_VOLTAGE 3.3 [current_design]
set_property CFGBVS VCCO [current_design]

```
  
  ## Simulation: Waveforms for all required scenarios and analysis of expected vs. observed results
  
  
#### Simulation Waveform
  <img width="975" height="383" alt="image" src="https://github.com/user-attachments/assets/b2333f11-ed63-4c3c-b1d4-46d9beab52c9" />

The testbench applies five input combinations, holding each for 20 ns. The STATUS bus is displayed in decimal: 0 = "00", 1 = "01", 2 = "10", 3 = "11".

| Time (ns) | D | K | P | S | W | STATUS | Result |
|---|---|---|---|---|---|---|---|
| 0–20 | 1 | 0 | 0 | 0 | 0 | 0 (00) | No warning, key out ✅ |
| 20–40 | 0 | 1 | 0 | 0 | 1 | 1 (01) | Door warning ✅ |
| 40–60 | 1 | 1 | 1 | 0 | 1 | 2 (10) | Seat-belt warning ✅ |
| 60–80 | 0 | 1 | 1 | 0 | 1 | 3 (11) | Both warnings ✅ |
| 80–100 | 1 | 1 | 1 | 1 | 0 | 0 (00) | No warning, belt fastened ✅ |


## Hardware: Elaborated and synthesized schematics, implementation, utilization and Basys3 behaviour


#### Elaborated Design
  <img width="975" height="270" alt="image" src="https://github.com/user-attachments/assets/cdd4d2db-ac1b-435b-91f9-7e99d0630775" />


*The elaborated schematic is Vivado's gate-level view of the VHDL before any optimization*

**RTL_AND** gates produce door_warning (**K·D’**) and belt_warning (**K·P·S’**).\
**RTL_OR** (W_i) combines the two warnings to drive the output W.\
**RTL_EQ** comparators and two **RTL_MUX** blocks form the when…else statement. The comparators check each condition, and the multiplexers select “11”, “10”, “01” or “00” for **STATUS[1:0]**, in priority order.


Synthesized Design


  <img width="975" height="459" alt="image" src="https://github.com/user-attachments/assets/9ecd0e24-f69f-48cd-a855-a1f83630035e" />

*The synthesized schematic shows how Vivado mapped the design onto real FPGA resources:*

- **IBUF / OBUF:** 4 input buffers (D, K, P, S) and 3 output buffers (W, STATUS[0], STATUS[1]) connect the logic to the physical pins.
- **LUT4** (`W_OBUF_inst_i_1`) uses all four inputs to compute **W = K(D' + PS')**.
- **LUT2** (`STATUS_OBUF[0]_inst_i_1`) uses D and K to compute the door warning, **STATUS[0] = K·D'**.
- **LUT3** (`STATUS_OBUF[1]_inst_i_1`) uses K, P and S to compute the belt warning, **STATUS[1] = K·P·S'**.

Compared with the elaborated design, the comparators and multiplexers from the `when…else` statement are gone. Synthesis optimized everything into 3 look-up tables (10 cells, 7 I/O ports, 14 nets in total).

  
#### Implemented Design


  <img width="975" height="486" alt="image" src="https://github.com/user-attachments/assets/c2ad4230-cf33-4320-b67a-cdadadb707a5" />


*The implemented design shows where the logic was placed on the physical Artix-7 chip:*

The device view shows the FPGA divided into six clock regions (X0Y0 to X1Y2). The design is so small that only a few highlighted cells are used. Almost the whole chip stays empty. The design was successfully placed and routed.

  
#### Utilization report


  <img width="758" height="181" alt="image" src="https://github.com/user-attachments/assets/103d03f8-584f-48cf-aad6-ceec00071bc5" />


*The utilization report confirms how little of the FPGA the design needs:*

| Resource | Used | Available | Utilization |
|---|---|---|---|
| Slice LUTs | 2 | 20,800 | ~0.01 % |
| Slices | 2 | 8,150 | ~0.02 % |
| Bonded IOB | 7 | 106 | ~6.6 % |

The 7 IOBs match the 7 ports (4 switches + 3 LEDs). The schematic shows 3 LUTs, but the report counts only 2. Each physical LUT in the Artix-7 is a 6-input LUT that can hold two smaller functions, so the LUT2 and LUT3 can share one LUT.



## Reflection: Challenges/resolutions and learning linked to theory

