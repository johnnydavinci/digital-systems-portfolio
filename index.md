# Digital Systems Design Portfolio

**Name:** Quang Nguyen 
**Course/Section:** Adv Digital & Embedded Systems - ELIC-329-0NA

This portfolio documents four digital systems design projects, including design, VHDL source code, simulation verification, FPGA implementation and reflection.

## Project Reports

1. [Project 1 — Car seat belt warning light](projects/project1.html)
   
  **Introduction: idea and objectives**
  
  Idea: A digital control circuit that turns on a car’s seat-belt warning light (L) based on four sensor inputs. A sensor sets its output to 1 when its condition is true; for example, the pressure sensor outputs 1 when someone is in the seat.
  
  Input	= 1 when…	Basys3 switch / pin
  D (door_closed)	door is closed	SW0 / V17
  K (key_in)	key is in the ignition	SW1 / V16
  P (seat_pressure)	a person is in the seat	SW2 / W16
  S (seat_belt)	seat belt is fastened	SW3 / W17
  L (output)	warning light is ON	LED0 / U16
  
  Objectives (from the handout):
  
  Design the circuit the traditional way: truth table, K-map and Boolean simplification, then build it in Multisim. (Part I)
  Implement the same circuit on the Basys3 FPGA (Artix-7 xc7a35tcpg236-1) through the Vivado flow: design entry, simulation, synthesis, implementation, bitstream and programming. (Part II)
  Challenge: extend the design so a 2-bit STATUS output tells apart a door warning and a seat-belt warning. (Part III)
  
  
  Design: Explanation of design and labelled block diagram
  Code: Dexplanation of code blocks or sections (key parts), Link to the design source, testbench and relevant XDC
  Simulation: Waveforms for all required scenarios and analysis of expected vs. observed results
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
