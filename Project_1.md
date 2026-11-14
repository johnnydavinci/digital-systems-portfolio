<h1 align="center">💡Digital Systems Design Portfolio💡</h1>

**Name:** Quang Nguyen 

**Course/Section:** Adv Digital & Embedded Systems - ELIC-329-0NA

This portfolio documents four digital systems design projects, including design, VHDL source code, simulation verification, FPGA implementation and reflection.

<h2 align="center">💻Project Reports💻</h2>

<h3> Project 1 – Car Seat Belt Warning Light 🚗</h3>
   
  #### Introduction
  
  **Idea**: A digital control circuit that turns on a car’s seat-belt warning light (L) based on four sensor inputs. A sensor sets its output to 1 when its condition is true; for example, the pressure sensor outputs 1 when someone is in the seat.
  
| Input | = 1 when… | Basys3 switch / pin |
|---|---|---|
| D (door_closed) | door is closed | SW0 / V17 |
| K (key_in) | key is in the ignition | SW1 / V16 |
| P (seat_pressure) | a person is in the seat | SW2 / W16 |
| S (seat_belt) | seat belt is fastened | SW3 / W17 |
| W (output) | warning light is ON | LED0 / U16 |
| STATUS[1:0] | warning type | LED2 / U19, LED1 / E19 |
  
  **Objectives** (from the handout):
  
  1. Design the circuit the traditional way: truth table, K-map and Boolean simplification. (Part I)
  2. Implement the same circuit on the Basys3 FPGA (Artix-7 xc7a35tcpg236-1) through the Vivado flow: design entry, simulation, synthesis,              implementation, bitstream and programming. (Part II)
  3. Challenge: extend the design so a 2-bit STATUS output tells apart a door warning and a seat-belt warning. (Part III)
  
  
  #### Design

Truth Table


| D | K | P | S | Minterm |
|---|---|---|---|---------|
| 0 | 1 | 0 | 0 | 4 |
| 0 | 1 | 0 | 1 | 5 |
| 0 | 1 | 1 | 0 | 6 |
| 0 | 1 | 1 | 1 | 7 |
| 1 | 1 | 1 | 0 | 14 |

K-map


| DK \ PS | 00 | 01 | 11 | 10 |
|---------|----|----|----|----|
| **00**  | 0  | 0  | 0  | 0  |
| **01**  | 1  | 1  | 1  | 1  |
| **11**  | 0  | 0  | 0  | 1  |
| **10**  | 0  | 0  | 0  | 0  |

Boolean Simplification


`L = D'K + KPS' = K(D' + PS')`

  
  #### Code
  

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
  
  **Simulation: Waveforms for all required scenarios and analysis of expected vs. observed results**
  
  
Simulation Waveform
  <img width="975" height="383" alt="image" src="https://github.com/user-attachments/assets/b2333f11-ed63-4c3c-b1d4-46d9beab52c9" />

  

#### Hardware: Elaborated and synthesized schematics, implementation, utilization and Basys3 behaviour



Elaborated Design 


  <img width="975" height="270" alt="image" src="https://github.com/user-attachments/assets/cdd4d2db-ac1b-435b-91f9-7e99d0630775" />

Synthesized Design


  <img width="975" height="459" alt="image" src="https://github.com/user-attachments/assets/9ecd0e24-f69f-48cd-a855-a1f83630035e" />

  
Implemented Design


  <img width="975" height="486" alt="image" src="https://github.com/user-attachments/assets/c2ad4230-cf33-4320-b67a-cdadadb707a5" />

  
Utilization report


  <img width="758" height="181" alt="image" src="https://github.com/user-attachments/assets/103d03f8-584f-48cf-aad6-ceec00071bc5" />



#### Reflection: Challenges/resolutions and learning linked to theory

