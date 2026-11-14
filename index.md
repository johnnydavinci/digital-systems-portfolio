<h1 align="center">💡Digital Systems Design Portfolio💡</h1>

**Name:** Quang Nguyen 

**Course/Section:** Adv Digital & Embedded Systems - ELIC-329-0NA

This portfolio documents four digital systems design projects, including design, VHDL source code, simulation verification, FPGA implementation and reflection.

<h2 align="center">💻Project Reports💻</h2>

<h3> Project 1 – Car Seat Belt Warning Light 🚗</h3>
   
  #### Introduction: idea and objectives
  
  **Idea**: A digital control circuit that turns on a car’s seat-belt warning light (L) based on four sensor inputs. A sensor sets its output to 1 when its condition is true; for example, the pressure sensor outputs 1 when someone is in the seat.
  
  Input	= 1 when…	Basys3 switch / pin
  D (door_closed)	door is closed	SW0 / V17
  K (key_in)	key is in the ignition	SW1 / V16
  P (seat_pressure)	a person is in the seat	SW2 / W16
  S (seat_belt)	seat belt is fastened	SW3 / W17
  L (output)	warning light is ON	LED0 / U16
  
  **Objectives** (from the handout):
  
  1. Design the circuit the traditional way: truth table, K-map and Boolean simplification. (Part I)
  2. Implement the same circuit on the Basys3 FPGA (Artix-7 xc7a35tcpg236-1) through the Vivado flow: design entry, simulation, synthesis,              implementation, bitstream and programming. (Part II)
  3. Challenge: extend the design so a 2-bit STATUS output tells apart a door warning and a seat-belt warning. (Part III)
  
  
  **Design: Explanation of design and labelled block diagram**

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
  
  **Code: Dexplanation of code blocks or sections (key parts), Link to the design source, testbench and relevant XDC**

  
  **Simulation: Waveforms for all required scenarios and analysis of expected vs. observed results**

Elaborated Design 
  <img width="975" height="270" alt="image" src="https://github.com/user-attachments/assets/cdd4d2db-ac1b-435b-91f9-7e99d0630775" />
Simulation Waveform
  <img width="975" height="383" alt="image" src="https://github.com/user-attachments/assets/b2333f11-ed63-4c3c-b1d4-46d9beab52c9" />
Synthesized Design
  <img width="975" height="459" alt="image" src="https://github.com/user-attachments/assets/9ecd0e24-f69f-48cd-a855-a1f83630035e" />
Implemented Design
  <img width="975" height="486" alt="image" src="https://github.com/user-attachments/assets/c2ad4230-cf33-4320-b67a-cdadadb707a5" />
Utilization report
  <img width="758" height="181" alt="image" src="https://github.com/user-attachments/assets/103d03f8-584f-48cf-aad6-ceec00071bc5" />



  Hardware: Elaborated and synthesized schematics, implementation, utilization and Basys3 behaviour
  Reflection: Challenges/resolutions and learning linked to theory


2. [Project 2 — Add title](projects/project2.html)

Introduction: Project idea and objectives
Design: Explanation of design and labelled block diagram
Code: Dexplanation of code blocks or sections (key parts), Link to the design source, testbench and relevant XDC
Simulation: Waveforms for all required scenarios and analysis of expected vs. observed results
Hardware: Elaborated and synthesized schematics, implementation, utilization and Basys3 behaviour
Reflection: Challenges/resolutions and learning linked to theory

3. [Project 3 — Add title](projects/project3.html)

Introduction: Project idea and objectives
Design: Explanation of design and labelled block diagram
Code: Dexplanation of code blocks or sections (key parts), Link to the design source, testbench and relevant XDC
Simulation: Waveforms for all required scenarios and analysis of expected vs. observed results
Hardware: Elaborated and synthesized schematics, implementation, utilization and Basys3 behaviour
Reflection: Challenges/resolutions and learning linked to theory

4. [Project 4 — Add title](projects/project4.html)

Introduction: Project idea and objectives
Design: Explanation of design and labelled block diagram
Code: Dexplanation of code blocks or sections (key parts), Link to the design source, testbench and relevant XDC
Simulation: Waveforms for all required scenarios and analysis of expected vs. observed results
Hardware: Elaborated and synthesized schematics, implementation, utilization and Basys3 behaviour
Reflection: Challenges/resolutions and learning linked to theory


> Replace all bracketed placeholders with your own content. See the repository README for instructions.
