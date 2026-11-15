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

*Top-level structural design.* It does not describe any logic itself; it connects the three modules:
- `debouncer_inst` cleans the `btn_count` press and outputs `debounced_btn`.
- `counter_inst` uses `debounced_btn` as its clock, so it counts once per press. `btn_reset` clears it.
- `decoder_inst` turns the 4-bit count into the 7-segment pattern and sets `anode`.
- `leds <= count_sig` also shows the count in binary on LD0–LD3.


**button_debounce.vhd**
````

library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity button_debounce is
    generic (
        STABLE_CYCLES : positive := 2000000
    );
    port (
        clk_in : in  STD_LOGIC;
        reset  : in  STD_LOGIC;
        btn_in : in  STD_LOGIC;
        btn_db : out STD_LOGIC
    );
end button_debounce;

architecture Behavioral of button_debounce is

    signal btn1, btn2 : STD_LOGIC := '0';
    signal btn_stable : STD_LOGIC := '0';

    signal count : integer range 0 to STABLE_CYCLES := 0;

begin

    process(clk_in, reset)
    begin

        if reset = '1' then
            btn1 <= '0';
            btn2 <= '0';
            btn_stable <= '0';
            count <= 0;

        elsif rising_edge(clk_in) then

            -- Read button input into internal signals
            btn1 <= btn_in;
            btn2 <= btn1;

            -- If the button matches the accepted state,
            -- there is no new press or release to check.
            if btn2 = btn_stable then
                count <= 0;

            else
                -- The button has changed, count how long it stays at the new value.
                if count < STABLE_CYCLES - 1 then
                    count <= count + 1;

                else
                    -- The new value has remained stable for enough cycles
                    btn_stable <= btn2;
                    count <= 0;

                end if;
            end if;

        end if;
    end process;

    -- Send the clean button signal to the counter.
    btn_db <= btn_stable;

end Behavioral;
````


*Removes mechanical bounce from the push button.*
- **Synchronizer (`btn1`, `btn2`):** The button signal passes through two flip-flops so it lines up safely with the 100 MHz clock.
- **Stability counter:** When the button differs from the accepted state (`btn_stable`), `count` increases every clock cycle. If the button bounces back, the count resets to 0.
- **Accept new value:** Only after the button stays the same for `STABLE_CYCLES` cycles does `btn_stable` change. With `STABLE_CYCLES = 2_000_000` this is 20 ms.
- **Generic:** `STABLE_CYCLES` can be changed without editing the logic. It was set to 10 (100 ns) for simulation so the testbench runs quickly, and to 2,000,000 (20 ms) for the Basys3 board.
- **Reset:** Asynchronous, like the counter. It clears all internal signals immediately.


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

*Generic N-bit up-counter (N = 4 here, so 0 to F).*
- `reset = '1'` clears the count immediately. This is an **asynchronous reset**: it is checked before the clock edge.
- On each `rising_edge(clk)` the count increases by 1. After `1111` it wraps back to `0000` automatically.
- The count is stored as `unsigned` so `+ 1` works, then converted to `std_logic_vector` for the output.


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

*Hex-to-7-segment decoder.*
- A `case…when` statement maps each 4-bit value (0–F) to a 7-bit `gfedcba` pattern.
- The display is **active-low**: `0` = segment ON. For example, `"1000000"` lights every segment except g, showing **0**.
- `anode <= "1110"` turns on only the rightmost digit.


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

*Maps every port to a Basys3 pin:* the clock to W5 (100 MHz, 10 ns period), `btn_count` to the centre button (U18), `btn_reset` to the up button (T18), `leds[3:0]` to LD0–LD3, `sevenseg[6:0]` to segments a–g and `anode[3:0]` to the four digit-select pins. All pins use LVCMOS33 (3.3 V).

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


*The testbench generates a 100 MHz clock and tests four cases:*
1. Reset at the start.
2. 18 short presses: the count goes 0 → F, wraps to 0, then reaches 2.
3. One long hold: the count increases only once (2 → 3).
4. Reset in the middle, then two more presses: 0 → 1 → 2.

The debounce time was shortened only for simulation, so 2 µs presses are accepted.


<img width="1060" height="466" alt="image" src="https://github.com/user-attachments/assets/2c800eb0-4820-4bc4-903a-e8c788d7d7a6" />


*Result:* `leds` counts 1, 2 … F, wraps to 0, then 1, 2. The long press gives only one count (3). Reset returns it to 0, and counting resumes (1, 2). `sevenseg` changes with the count; for example `79` is the pattern for "1" and `07` is "F". `anode` stays at `e` (`1110`), so only the rightmost digit is used. All results match the expected behaviour.


## Hardware


**Elaborated Design**\
<img width="1207" height="392" alt="image" src="https://github.com/user-attachments/assets/2f450fa9-7ab5-4f21-9edb-20257f6cff11" />


*Shows the structure from the VHDL:* the three blocks `debouncer_inst`, `counter_inst` and `decoder_inst` are wired exactly as in the block diagram. `btn_reset` goes to both the debouncer and the counter, and the debounced output (`btn_db`) feeds the counter's `clk`. `anode` is a constant `"1110"` sent through an output buffer.


**Synthesized Design**\
<img width="1420" height="944" alt="image" src="https://github.com/user-attachments/assets/d461b1c3-68c8-41da-a484-ce64946dc72a" />


*Shows the design mapped to FPGA resources:* input buffers (IBUF) for the buttons and clock, a global clock buffer (BUFG) for the 100 MHz clock, the three modules in the middle, and output buffers (OBUF) for all 15 output pins (4 LEDs, 7 segments, 4 anodes).


**Implemented Design**\
<img width="268" height="425" alt="image" src="https://github.com/user-attachments/assets/2ea391bd-3333-481d-994c-364e53c81835" />


*Shows where the logic was placed on the Artix-7 chip.* Only a few small highlighted areas are used, near the I/O edges where the switches, buttons and display pins connect. The rest of the chip is empty.


**Utilization report**\
<img width="794" height="211" alt="image" src="https://github.com/user-attachments/assets/8c9893b3-0f1b-49b2-9624-4641450a765b" />


| Resource | Used | Available |
|---|---|---|
| Slice LUTs | 11 | 20,800 |
| Slice Registers (flip-flops) | 11 | 41,600 |
| Bonded IOB | 18 | 106 |
| BUFGCTRL (clock buffer) | 1 | 32 |

Unlike Project 1, this design uses **registers**, because counters and the debouncer are sequential and must remember their state. The 4-bit counter uses 4 registers, the debouncer 7, and the decoder 0 (it is purely combinational). The 18 IOBs are 1 clock + 2 buttons + 4 LEDs + 7 segments + 4 anodes.


**Basys3 behaviour**


After programming, pressing the centre button increased the digit on the rightmost display by one each time (0 → F, then back to 0), and LD0–LD3 showed the same value in binary. Holding the button counted only once. Pressing the up button reset the display to 0.

## Reflection

**Challenges and resolutions**
- **Simulation didn't count:** The debouncer needs the button to stay stable for 20 ms (2,000,000 clock cycles), so a few microseconds of simulation showed no change. I set `STABLE_CYCLES` to 10 (100 ns) for simulation only, then changed it back to 2,000,000 for the Basys3, because 100 ns is far too short for a real button.
- **Port mismatch:** The component declaration in the top level had to match the debouncer's real ports, including `reset`. Otherwise elaboration failed.
- **Reading the display values:** The 7-segment display is active-low, so the bit patterns look "inverted" (`1000000` = 0). Checking them against the truth table confirmed the decoder.

**Learning linked to theory**
- **Sequential vs. combinational logic:** The counter and debouncer use flip-flops to remember their state, while the decoder is purely combinational. The utilization report confirms this: the decoder uses 0 registers.
- **Asynchronous reset:** The reset is checked before `rising_edge(clk)`, so it works immediately without waiting for a clock edge.
- **Switch bounce:** A mechanical button creates many fast on/off transitions. Without debouncing, one press could count several times.
- **Structural VHDL and generics:** The top level reuses modules as components. `generic map (N => 4)` sets the counter size and `STABLE_CYCLES` sets the debounce time, without changing either module's code.
