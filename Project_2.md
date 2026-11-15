<h1 align="center">💻 Project Reports 💻
<br>
<h1 align="center"> Project 2 🔢 Event Counter</h1>

<img width="2752" height="1536" alt="Gemini_Generated_Image_o4hitro4hitro4hi" src="https://github.com/user-attachments/assets/545356cb-f6b7-4f23-8bb1-227d5376ebad" />


## Introduction

**Idea** 

A counter whose value is shown on the Basys3 seven-segment display and LEDs. It is built in two parts:

- **Part I – Counter with display:** The counter increases automatically once every second (1 Hz) and counts 0 to F in hex.
- **Part II – Button event counter:** The counter increases once each time a push button is pressed, and a second button resets it to 0.

**Objectives** 

1. Design and implement sequential circuits in VHDL, including modulo-M counters and button event counters.
2. Use a configurable frequency divider with VHDL `generic` and `generic map`.
3. Implement and test the counter on the Basys3 FPGA using push buttons and the 7-segment display.

## Design

**Block Diagram**

- **Part I:** 100 MHz clock → `clk_div` (1 Hz) → `modMcounter` (4-bit) → `Hex_7seg` → 7-segment display + LEDs
- **Part II:** Push button → `button_debounce` → `modMcounter` (4-bit) → `Hex_7seg` → 7-segment display + LEDs

**Components**

| Module | Function |
|---|---|
| `clk_div` | Divides the 100 MHz board clock down to a 1 Hz slow clock (Part I only) |
| `modMcounter` | Generic modulo-M counter with asynchronous active-high reset. Set to 4 bits, so it counts 0–15 (0–F), since 2⁴ = 16 |
| `Hex_7seg` | Converts the 4-bit count into the 7 segment signals using a `case…when` statement |
| `button_debounce` | Filters out button bounce. The output changes only after the button stays stable for about 20 ms (Part II only, provided) |
| `count_display` | Top-level design that connects all modules using structural VHDL |

**Key design points**

- The Basys3 seven-segment display is **common anode (active-low)**: `0` turns a segment ON and `1` turns it OFF.
- Only the rightmost digit is used, so `anode = "1110"`.
- In Part II, the debounced button signal replaces the 1 Hz clock as the counter's clock, so the clock divider is not needed.


## Code


**Count-display.vhd**
````
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity Count_display is
    Port ( clock     : in STD_LOGIC;
           btn_count : in STD_LOGIC;
           btn_reset : in STD_LOGIC;
           leds      : out STD_LOGIC_VECTOR (3 downto 0);
           sevenseg  : out STD_LOGIC_VECTOR (6 downto 0);
           anode     : out STD_LOGIC_VECTOR (3 downto 0));
end Count_display;

architecture Structural of Count_display is
    component button_debounce
        Port ( clk_in : in STD_LOGIC;   -- Corrected to clk_in
               btn_in : in STD_LOGIC;
               btn_db : out STD_LOGIC);
    end component;

    component modMcounter
        generic ( N : integer := 4 );
        port ( clk   : in STD_LOGIC;
               reset : in STD_LOGIC;
               count : out STD_LOGIC_VECTOR(N-1 downto 0));
    end component;

    component Hex_7seg
        port ( hexdigit : in STD_LOGIC_VECTOR(3 downto 0);
               seg      : out STD_LOGIC_VECTOR(6 downto 0);
               anode    : out STD_LOGIC_VECTOR(3 downto 0));
    end component;

    signal debounced_btn : STD_LOGIC;
    signal count_sig     : STD_LOGIC_VECTOR(3 downto 0);
begin

    -- Debounce the physical push button
    debouncer_inst: button_debounce
        port map(
            clk_in => clock,            -- Corrected to clk_in
            btn_in => btn_count,
            btn_db => debounced_btn
        );

    -- Counter triggered by the debounced button press
    counter_inst: modMcounter
        generic map ( N => 4 )
        port map(
            clk => debounced_btn,
            reset => btn_reset,
            count => count_sig
        );

    -- Decode the counter output for the 7-segment display
    decoder_inst: Hex_7seg
        port map(
            hexdigit => count_sig,
            seg => sevenseg,
            anode => anode
        );

    -- Wire the 4-bit count to the discrete LEDs
    leds <= count_sig;

end Structural;
````


**button_debounce.vhd**
````
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity Count_display is
    Port ( clock     : in STD_LOGIC;
           btn_count : in STD_LOGIC;
           btn_reset : in STD_LOGIC;
           leds      : out STD_LOGIC_VECTOR (3 downto 0);
           sevenseg  : out STD_LOGIC_VECTOR (6 downto 0);
           anode     : out STD_LOGIC_VECTOR (3 downto 0));
end Count_display;

architecture Structural of Count_display is
    component button_debounce
        Port ( clk_in : in STD_LOGIC;
               reset  : in STD_LOGIC;   -- Added reset port to match debouncer file
               btn_in : in STD_LOGIC;
               btn_db : out STD_LOGIC);
    end component;

    component modMcounter
        generic ( N : integer := 4 );
        port ( clk   : in STD_LOGIC;
               reset : in STD_LOGIC;
               count : out STD_LOGIC_VECTOR(N-1 downto 0));
    end component;

    component Hex_7seg
        port ( hexdigit : in STD_LOGIC_VECTOR(3 downto 0);
               seg      : out STD_LOGIC_VECTOR(6 downto 0);
               anode    : out STD_LOGIC_VECTOR(3 downto 0));
    end component;

    signal debounced_btn : STD_LOGIC;
    signal count_sig     : STD_LOGIC_VECTOR(3 downto 0);
begin

    -- Debounce the physical push button
    debouncer_inst: button_debounce
        port map(
            clk_in => clock,
            reset  => btn_reset,      -- Connected to the top-level reset button
            btn_in => btn_count,
            btn_db => debounced_btn
        );

    -- Counter triggered by the debounced button press
    counter_inst: modMcounter
        generic map ( N => 4 )
        port map(
            clk => debounced_btn,
            reset => btn_reset,
            count => count_sig
        );

    -- Decode the counter output for the 7-segment display
    decoder_inst: Hex_7seg
        port map(
            hexdigit => count_sig,
            seg => sevenseg,
            anode => anode
        );

    -- Wire the 4-bit count to the discrete LEDs
    leds <= count_sig;

end Structural;
````


**modMcounter.vhd**
````
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity modMcounter is
    generic ( N : integer := 4 ); -- 4 bits to count 0 to F
    port (
        clk   : in STD_LOGIC;
        reset : in STD_LOGIC;
        count : out STD_LOGIC_VECTOR(N-1 downto 0)
    );
end modMcounter;

architecture Behavioral of modMcounter is
    signal count_sig : unsigned(N-1 downto 0) := (others => '0');
begin
    process(clk, reset)
    begin
        if reset = '1' then
            count_sig <= (others => '0');
        elsif rising_edge(clk) then
            count_sig <= count_sig + 1;
        end if;
    end process;
    
    count <= std_logic_vector(count_sig);
end Behavioral;
````


**Hex_7seg.vhd**
````
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity Hex_7seg is
    port (
        hexdigit : in STD_LOGIC_VECTOR(3 downto 0);
        seg      : out STD_LOGIC_VECTOR(6 downto 0);
        anode    : out STD_LOGIC_VECTOR(3 downto 0)
    );
end Hex_7seg;

architecture Behavioral of Hex_7seg is
begin
    -- 0 selects the digit, 1 disables it. "1110" enables only the rightmost digit.
    anode <= "1110";

    process(hexdigit)
    begin
        case hexdigit is
            when "0000" => seg <= "1000000"; -- 0
            when "0001" => seg <= "1111001"; -- 1
            when "0010" => seg <= "0100100"; -- 2
            when "0011" => seg <= "0110000"; -- 3
            when "0100" => seg <= "0011001"; -- 4
            when "0101" => seg <= "0010010"; -- 5
            when "0110" => seg <= "0000010"; -- 6
            when "0111" => seg <= "1111000"; -- 7
            when "1000" => seg <= "0000000"; -- 8
            when "1001" => seg <= "0010000"; -- 9
            when "1010" => seg <= "0001000"; -- A
            when "1011" => seg <= "0000011"; -- B
            when "1100" => seg <= "1000110"; -- C
            when "1101" => seg <= "0100001"; -- D
            when "1110" => seg <= "0000110"; -- E
            when "1111" => seg <= "0000111"; -- F
            when others => seg <= "1111111"; -- All off
        end case;
    end process;
end Behavioral;
````


**Contranst**
````
## Clock signal (100 MHz)
set_property PACKAGE_PIN W5 [get_ports clock]
set_property IOSTANDARD LVCMOS33 [get_ports clock]
create_clock -add -name sys_clk_pin -period 10.00 -waveform {0 5} [get_ports clock]

## LEDs
set_property PACKAGE_PIN U16 [get_ports {leds[0]}]
set_property IOSTANDARD LVCMOS33 [get_ports {leds[0]}]
set_property PACKAGE_PIN E19 [get_ports {leds[1]}]
set_property IOSTANDARD LVCMOS33 [get_ports {leds[1]}]
set_property PACKAGE_PIN U19 [get_ports {leds[2]}]
set_property IOSTANDARD LVCMOS33 [get_ports {leds[2]}]
set_property PACKAGE_PIN V19 [get_ports {leds[3]}]
set_property IOSTANDARD LVCMOS33 [get_ports {leds[3]}]

## 7 segment display
set_property PACKAGE_PIN W7 [get_ports {sevenseg[0]}]
set_property IOSTANDARD LVCMOS33 [get_ports {sevenseg[0]}]
set_property PACKAGE_PIN W6 [get_ports {sevenseg[1]}]
set_property IOSTANDARD LVCMOS33 [get_ports {sevenseg[1]}]
set_property PACKAGE_PIN U8 [get_ports {sevenseg[2]}]
set_property IOSTANDARD LVCMOS33 [get_ports {sevenseg[2]}]
set_property PACKAGE_PIN V8 [get_ports {sevenseg[3]}]
set_property IOSTANDARD LVCMOS33 [get_ports {sevenseg[3]}]
set_property PACKAGE_PIN U5 [get_ports {sevenseg[4]}]
set_property IOSTANDARD LVCMOS33 [get_ports {sevenseg[4]}]
set_property PACKAGE_PIN V5 [get_ports {sevenseg[5]}]
set_property IOSTANDARD LVCMOS33 [get_ports {sevenseg[5]}]
set_property PACKAGE_PIN U7 [get_ports {sevenseg[6]}]
set_property IOSTANDARD LVCMOS33 [get_ports {sevenseg[6]}]

set_property PACKAGE_PIN U2 [get_ports {anode[0]}]
set_property IOSTANDARD LVCMOS33 [get_ports {anode[0]}]
set_property PACKAGE_PIN U4 [get_ports {anode[1]}]
set_property IOSTANDARD LVCMOS33 [get_ports {anode[1]}]
set_property PACKAGE_PIN V4 [get_ports {anode[2]}]
set_property IOSTANDARD LVCMOS33 [get_ports {anode[2]}]
set_property PACKAGE_PIN W4 [get_ports {anode[3]}]
set_property IOSTANDARD LVCMOS33 [get_ports {anode[3]}]

## Buttons
set_property PACKAGE_PIN U18 [get_ports btn_count]
set_property IOSTANDARD LVCMOS33 [get_ports btn_count]
set_property PACKAGE_PIN T18 [get_ports btn_reset]
set_property IOSTANDARD LVCMOS33 [get_ports btn_reset]

## Configuration options
set_property CONFIG_VOLTAGE 3.3 [current_design]
set_property CFGBVS VCCO [current_design]
````


## Simulation


**Testbench**
````
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;

ENTITY Count_display_tb IS
END Count_display_tb;

ARCHITECTURE behavior OF Count_display_tb IS 
    COMPONENT Count_display
    PORT(
         clock     : IN  std_logic;
         btn_count : IN  std_logic;
         btn_reset : IN  std_logic;
         leds      : OUT std_logic_vector(3 downto 0);
         sevenseg  : OUT std_logic_vector(6 downto 0);
         anode     : OUT std_logic_vector(3 downto 0)
        );
    END COMPONENT;

    signal clock     : std_logic := '0';
    signal btn_count : std_logic := '0';
    signal btn_reset : std_logic := '0';
    signal leds      : std_logic_vector(3 downto 0);
    signal sevenseg  : std_logic_vector(6 downto 0);
    signal anode     : std_logic_vector(3 downto 0);

    constant clock_period : time := 10 ns;
    constant press_time   : time := 2 us;   -- must be longer than the debounce time
    constant gap_time     : time := 2 us;
BEGIN
    uut: Count_display PORT MAP (
          clock     => clock,
          btn_count => btn_count,
          btn_reset => btn_reset,
          leds      => leds,
          sevenseg  => sevenseg,
          anode     => anode
        );

    clock_process : process
    begin
        clock <= '0';
        wait for clock_period/2;
        clock <= '1';
        wait for clock_period/2;
    end process;

    stim_proc: process
    begin
        -- 1) Reset at start
        btn_reset <= '1';
        wait for 100 ns;
        btn_reset <= '0';
        wait for 1 us;

        -- 2) 18 presses: count 0 -> F, wrap to 0, then 1 -> 2
        for i in 1 to 18 loop
            btn_count <= '1';
            wait for press_time;
            btn_count <= '0';
            wait for gap_time;
        end loop;

        -- 3) Long hold: should count only ONCE (2 -> 3)
        btn_count <= '1';
        wait for 10 us;
        btn_count <= '0';
        wait for gap_time;

        -- 4) Reset in the middle: count returns to 0
        btn_reset <= '1';
        wait for 1 us;
        btn_reset <= '0';
        wait for gap_time;

        -- 5) Two more presses after reset: 0 -> 1 -> 2
        for i in 1 to 2 loop
            btn_count <= '1';
            wait for press_time;
            btn_count <= '0';
            wait for gap_time;
        end loop;

        wait;
    end process;
END behavior;

````

<img width="1060" height="466" alt="image" src="https://github.com/user-attachments/assets/2c800eb0-4820-4bc4-903a-e8c788d7d7a6" />


## Hardware : Elaborated and synthesized schematics, implementation, utilization and Basys3 behaviour


**Elaborated Design**
<img width="1207" height="392" alt="image" src="https://github.com/user-attachments/assets/2f450fa9-7ab5-4f21-9edb-20257f6cff11" />


**Synthesized Design**
<img width="1420" height="944" alt="image" src="https://github.com/user-attachments/assets/d461b1c3-68c8-41da-a484-ce64946dc72a" />




Reflection: Challenges/resolutions and learning linked to theory
