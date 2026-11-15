<h3 align="center">🔢 Project 2 – Event Counter</h3>

#### Introduction

**Idea:** A counter whose value is shown on the Basys3 seven-segment display and LEDs. It is built in two parts:

- **Part I – Counter with display:** The counter increases automatically once every second (1 Hz) and counts 0 to F in hex.
- **Part II – Button event counter:** The counter increases once each time a push button is pressed, and a second button resets it to 0.

**Objectives** (from the handout):

1. Design and implement sequential circuits in VHDL, including modulo-M counters and button event counters.
2. Use a configurable frequency divider with VHDL `generic` and `generic map`.
3. Implement and test the counter on the Basys3 FPGA using push buttons and the 7-segment display.

#### Design

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
