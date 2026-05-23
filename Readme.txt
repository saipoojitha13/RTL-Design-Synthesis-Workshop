
-|Day1: RTL Simulation Basics|-

# What is RTL code & Spec?
## Why RTL Verification is Needed and role of simulation?
### What is Synthesis and how it converts the rtl into netlist?

------ What is RTL? ------
RTL design (Register Transfer Level) describes what hardware should do on every clock edge. 
RTL stands for:
•	Register → storage elements like flip-flops/registers 
•	Transfer → movement of data between registers 
•	Level → abstraction level used to describe hardware behavior 

In general, RTL can be treated as a bridge between an algorithmic idea (pseudo code/functional behavior) and real silicon hardware.
In the complete chip design flow, the design moves through multiple stages from specification to final silicon.
The process typically starts with a specification (spec) that defines the required functionality, performance, power, and area goals. 
Based on this specification, engineers write RTL code using HDLs such as Verilog or VHDL.

-----------RTL vs Other Abstraction Levels----------
RTL level = Register-to-register operations
Gate-Level = Actual gates and connections
Transistor-Level = MOSFET implementation

RTL is the most commonly used abstraction level in digital VLSI design.

What is a Specification (Spec)?
A specification is a document describing:
•	what the design should do 
•	inputs and outputs 
•	protocols 
•	timing behavior 
•	corner cases 
•	reset behavior 
•	error handling 

Example Specification for a Counter
- 4-bit counter
- increments every clock cycle
- synchronous reset
- wraps from 15 → 0
________________________________________
Why RTL Design Needs Verification?
RTL design may violate the specification due to:
•	logic mistakes 
•	timing mistakes 
•	FSM errors 
•	protocol violations 
Therefore, RTL must be verified before manufacturing. Here simulator and simulation comes into picture. Simulator is a tool that perform simulation on given RTL design with using testbench.
Simulation is used to execute RTL behavior over time and verify functionality using a testbench.
________________________________________
What is a Testbench?
A testbench is verification code used only for testing the DUT (Design Under Test).
Responsibilities of Testbench
•	generate clock/reset 
•	apply stimulus (inputs) 
•	monitor outputs 
•	compare expected vs actual behavior 

________________________________________
What is a Simulator?
A simulator is a tool that runs the RTL design and testbench together.
It evaluates the output on how input signals changes over time.
RTL + Testbench
       ↓
    Simulator
       ↓
Signal changes over time
       ↓
Outputs are evaluated
________________________________________

Simple Testbench Setup
Testbench signals
      ↓
DUT inputs
      ↓
RTL logic
      ↓
DUT outputs
      ↓
Observed by testbench
________________________________________

A Simple Icarus Verilog Simulation Flow
Icarus Verilog (iverilog) is an open-source Verilog simulator used for RTL simulation.
Simulation Flow
RTL + Testbench
       ↓
iverilog tool
(compile + elaborate)
       ↓
Creates executable file (a.out)
       ↓
Run executable
(./a.out or vvp a.out)
       ↓
Waveform dump (.vcd)
       ↓
GTKWave(tool used for displaying the waveform
(view waveforms).
--------------------------------------------------------------------------------------------

-->Points to observed while dealing with RTL code and testbench:

RTL modules represent hardware blocks. At an higher SOC level we have a chain of RTL modules interconnected with each to achieve the functionality. So, they usually need ports to communicate with external hardware/modules.
For ex:
module adder(
    input  [3:0] a,b,
    output [4:0] sum
);

Here a,b are primary inputs and sum was primary output.
In testbench no primary inputs/outputs are usually needed. Because every thing that needed for simulating the rtl were generated at the inside of testbench like input stimulus, driving DUT inputs, and monitor DUT outputs. So Nothing exists outside the testbench in simulation.


-----------
Synthesizer
-----------

When our design meets our expectation in simulation. Then we move to synthesis.
Synthesis is a process where the RTL design code was converted into netlist of standard cells.
Yosys is a open-source synthesis tool and can use perform synthesis on current RTL design

The synthesis flow mainly consists of:
1. Parsing = Reads and checks RTL syntax and builds internal design representation
	Example command: read_verilog design.v
2. Elaboration = Resolves modules hierarchy and connects instantiated modules
3. Optimization= done to improve area, timing, and power requirements. 
	This can be done by remove redundant logic, simplify Boolean equations, resize cells and restructure logic paths depending upon the necessity.
Constraints guide the synthesis tool during optimization and timing analysis. so that the synthesis tool chooses faster cells, larger drive strengths, different logic structures. 
Timing analysis is performed across different PVT (Process, Voltage, Temperature) corners to ensure reliable circuit operation under different manufacturing and environmental conditions.
This is necessary because transistor characteristics and delays like (ex:RC delays) change with process variations, supply voltage, and temperature, which directly affect circuit timing and performance.
4. Technology Mapping = Maps generic logic into standard cells
5. Netlist Generation = Writes synthesized gate-level netlist
	Example Command: write_verilog netlist.v

Below are some useful commands and their explanation:
read_verilog = Read RTL
read_liberty = Load standard cells
hierarchy = Resolve modules
synth = Run synthesis
abc = Optimize + map cells(here it choose actual cells)
write_verilog = Generate netlist
Example of usage:
read_verilog counter.v
read_liberty -lib skyxyz.lib
hierarchy -top counter
synth -top counter
abc -liberty skyxyz.lib
write_verilog counter_netlist.v

RTL vs Gate-Level Netlist Functional Equivalence:
The primary inputs and outputs of the design generally remain the same after RTL is synthesized into a gate-level netlist.
Because the interface remains unchanged, the same testbench can usually be used to verify both:
1. RTL design
2. Synthesized gate-level netlist
In both cases, the outputs should match for the same set of inputs and clock conditions. This confirms that synthesis preserved the intended functionality of the RTL design. 

Simple terms of synthesis:
During synthesis, RTL code is combined with standard-cell liberty libraries (.lib) and constraints.
The synthesis tool uses:
- RTL → to understand functionality
- Liberty library → to understand available hardware cells and timing information
- Constraints → to optimize timing, area, and power

Finally, The synthesis tool generates a gate-level netlist consisting of technology-specific standard cells such as logic gates and flip-flops.

---> Practical Tasks Performed <------

- Simulated RTL using Icarus Verilog
- Generated VCD dump file
- Viewed waveforms using GTKWave
- Performed synthesis using Yosys