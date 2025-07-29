# Utilization Factor and Aspect Ratio

Standard cells may not occupy the entirety of a chip footprint The % of chip that contains standard cells->1 unit NAND/Gates/etc is 1 sq units of the silicon wafer

![](imagepng)

Aspect ratio is the length vs width of the chip size

![](image-1png)

## 13 - Concept of Pre-placed Cells

Pre placed cells are logic modules that are portable and can be reused in many places Their locations within the silicon is fixed due to constraints like power distribution, routing optimization, etc

These cells can be further abstracted through black boxing, in which only their IO is exposed to the circuit designer, to protect foundry IP or simplify the simulation process

![](image-2png)

## 14 - De-coupling Capacitors

Decoupling capacitors are placed next to the pre-placed cells, to avoid voltage drop across a wire to preserve power signal

![](image-3png)

## 15 - Power Planning

### Ground Bounce

When a lot of capacitors discharge at once, ground bounce can occur This in turn can trigger voltage drops due to not enough current to charge all the capacitors that are discharging This issue can be avoided by having multiple power sources/rails to create a capacitor charging networks

## 16 - Pin Placement and Logical Cell Placement Blockage

We place the pins on the left and right of the core
Then, blockers are placed to avoid autoplacement of cells onto the pins:

![](image-4png)

## 20 - Netlist Binding and Initial Place Design

Netlist binding means merging the netlist, the abstraction of logic connectivity with physical cells

Placement physically places the blocks from the library onto the rows of the chip

![](image-8png)

## 21 - Optimize Placement Using Estimated Wire-length and Capacitance

Simulations tell us the optimal wirelength for connecting modules Repeaters can be added in line to maintain consistent signalling along the wire

## 22 - Final Placement Optimization

![](image-9png)

## 25 - Inputs for Cell Design Flow

### Standard Cells

Standard cells are predefined modules like NAND, XOR, AND gates etc

They can come in multiple sizes with different drive strengths which have different voltage thresholds:

![](image-11png)

### Cell Design Flow

#### Inputs

- Process design kits—allows designers to see what technologies are available/their physical sizes
- DRC/LVS rules check manufacturability
- SPICE models help simulate cell performance
- Library/user defined specs such as clock speed, area budget, power constraints, IO, metal layers, etc

## 26 - Circuit Design Step

### Process:

**Functional Implementation:** The basic function is implemented using a combination of PMOS (pull-up network) and NMOS (pull-down network) transistors

**Sizing (W/L Ratios):** The Width-to-Length (W/L) ratios of the individual PMOS and NMOS transistors are determined This is a critical step for optimization

The sizing directly influences the drive strength of the cell, its switching threshold, and its power consumption

A common goal is to achieve a specific PMOS:NMOS ratio to ensure symmetrical rise and fall times, which is crucial for predictable timing For instance, a common ratio is W/L of PMOS being 25 times the W/L of NMOS

**Meeting Specifications:** The design must adhere to user-defined specifications, such as a target switching threshold (eg, 12V) or a specific drive strength These specifications dictate the required transistor sizing

### Inputs:

- **Spice Models:** Provided by the foundry, these models contain detailed electrical parameters of the transistors (eg, threshold voltage, mobility)
- **User-Defined Specifications:** These are the performance targets for the cell, such as cell height, drive strength, and required supply voltage

### Output:

- **Circuit Description Language (CDL):** The final output is a spice netlist, typically in CDL format This file describes the transistors and their interconnections, along with their specified W/L ratios, which can then be used for simulation and layout design

## 27 - Layout Design Step

### Process:

**Euler Path Identification:** To create the most compact and efficient layout, an optimal ordering of the transistors is determined This is achieved by finding a common "Euler path" (or oilless path) that traverses the nodes in both the PMOS and NMOS network graphs only once

**Stick Diagram Creation:** Based on the Euler path, a stick diagram is drawn This is a simplified, cartoon-like representation of the layout that shows the relative placement of transistors and the routing of different layers (represented by colored lines)

**Layout Generation:** The stick diagram is converted into a full physical layout using a layout editor tool (like Magic) During this process, the designer must adhere strictly to the DRC rules provided by the foundry to ensure manufacturability

### Inputs:

- **Circuit Design (CDL file):** The transistor-level netlist from the previous step
- **Process Design Kit (PDK):** Contains DRC rules, layer information, and other foundry-specific data
- **User-Defined Specifications:** Includes requirements like cell height (determined by power rail separation) and pin locations

### Outputs:

- **GDSII File:** A standard file format that contains the final geometric data of the layout for all layers
- **Extracted Spice Netlist:** After the layout is complete, parasitic capacitances and resistances from the physical wires and components are extracted This information is compiled into a new, more accurate spice netlist that represents the real-world behavior of the cell
- **LEF (Library Exchange Format) File:** Describes the physical abstract of the cell, including its dimensions, pin locations, and metal blockages, for use in automated place-and-route tools

## 28 - Typical Characterization Flow

### Key Inputs for Characterization Setup:

- **Spice Models:** Foundry provided models describing transistor behavior
- **Extracted Spice Netlist:** The netlist from the layout design step, which includes parasitic R and C values
- **Cell Behavior Recognition:** Defining the logical function of the cell (eg, inverter, NAND2)
- **Power Sources:** Defining the supply voltage (eg, VDD, VSS)
- **Stimulus:** Applying input waveforms with varying transition times (slew rates) to the cell's input pins
- **Output Load:** Connecting various load capacitances to the cell's output pins
- **Simulation Commands:** Specifying the type of analysis to be run (eg, transient analysis)

### The Process:

The characterization software (eg, Guna) uses all the above inputs to run numerous spice simulations

It simulates the cell's behavior across different PVT (Process, Voltage, Temperature) corners and for every combination of input slew and output load

### Outputs (The lib File):

- **Timing Models:** Contain data on cell delays and output transition times, typically presented in look-up tables indexed by input slew and output load
- **Power Models:** Contain data on leakage, internal, and switching power consumption

- **Noise Models:** Contain information on the cell's susceptibility to noise

## 29 - Timing Threshold Definitions

### Slew/Transition Thresholds:

Used to measure the transition time (slew rate) of a signal

- **slew_low_rise_threshold:** The lower percentage point (eg, 20% or 30% of VDD) on a rising edge
- **slew_high_rise_threshold:** The upper percentage point (eg, 80% of VDD) on a rising edge
- **slew_low_fall_threshold:** The lower percentage point on a falling edge
- **slew_high_fall_threshold:** The upper percentage point on a falling edge

### Delay Thresholds:

Used to measure the propagation delay from an input pin to an output pin

- **in_rise_threshold / in_fall_threshold:** The threshold point on the input waveform, typically 50% of VDD
- **out_rise_threshold / out_fall_threshold:** The threshold point on the output waveform, also typically 50% of VDD

Using standardized thresholds (eg, 50% for delay, 20%-80% for slew) ensures consistency

Incorrect threshold definition can lead to inaccurate timing measurements, such as impossible negative delays This happens when the chosen output threshold point is crossed chronologically before the input threshold point, often due to non-ideal waveform shapes caused by long wire delays

## 30 - Propagation Delay and Transition Time

### Propagation Delay:

**Definition:** The time it takes for a change at an input pin to propagate and cause a change at an output pin

**Calculation:** It is measured from the 50% point of the input transition to the 50% point of the output transition

**Formula:** Delay = Time(out_threshold) - Time(in_threshold)

**Types:**

- **Rise Delay:** When the output signal is rising (eg, for an inverter, this happens when the input is falling)
- **Fall Delay:** When the output signal is falling (eg, for an inverter, this happens when the input is rising)

### Signal Integrity and Repeaters:

Over long wires, signals can degrade (slew becomes very slow), which increases delay and makes timing unpredictable

To combat this, repeaters (buffers) are inserted along the long wire

The repeaters effectively break the long wire into shorter segments Each repeater receives the degraded signal, regenerates it into a sharp signal, and sends it down the next segment, thus maintaining signal integrity and controlling the overall delay
