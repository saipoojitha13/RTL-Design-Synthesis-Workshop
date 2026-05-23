**--> RTL Simulation & Synthesis Basics <--**


**------ What is RTL? ------**

-RTL design (Register Transfer Level) describes what hardware should do on every clock edge.

-RTL stands for:
1. Register → storage elements like flip-flops/registers
2. Transfer → movement of data between registers
3. Level → abstraction level used to describe hardware behavior

- In general, RTL can be treated as a bridge between an algorithmic idea (pseudo code/functional behavior) and real silicon hardware.
- In the complete chip design flow, the design moves through multiple stages from specification to final silicon.
The process typically starts with a specification (spec) that defines the required functionality, performance, power, and area goals.
- Based on this specification, engineers write RTL code using HDLs such as Verilog or VHDL.



**What is a Specification (Spec)?**
A specification is a document describing:
- what the design should do
- inputs and outputs
- protocols
- timing behavior
- corner cases
- reset behavior
- error handling

**Example Specification for a Counter**
- 4-bit counter
- increments every clock cycle
- synchronous reset
- wraps from 15 → 0



**Need of Testbench and Simulating the RTL design**
________________________________________
Why RTL Design Needs Verification?
RTL design may violate the specification due to:
1. logic mistakes
2. timing mistakes
3. FSM errors
4. protocol violations
- Therefore, RTL must be verified before manufacturing. Here simulator and simulation come into picture. Simulator is a tool that perform simulation on given RTL design with using testbench.
Simulation is used to execute RTL behavior over time and verify functionality using a testbench.
________________________________________

## What is a Testbench?

A testbench is verification code written specifically to test and validate the DUT (Design Under Test).  
It is usually written in Verilog or SystemVerilog and is used only for simulation purposes.

________________________________________
**What is a Simulator?** => A simulator is a tool that runs the RTL design and testbench together. It evaluates the output on how input signals changes over time.
```text
RTL Design + Testbench
            ↓
        Simulator
      (Icarus Verilog)
            ↓
     Signal Evaluation
            ↓
      VCD Waveform Dump
            ↓
          GTKWave
      (Waveform Viewer)
```

________________________________________


## Points to Observe While Dealing with RTL Code and Testbench

RTL modules represent actual hardware blocks.  
At a higher SoC level, multiple RTL modules are interconnected together to achieve the required functionality. Therefore, RTL modules usually require ports to communicate with external hardware blocks or other modules.

### Example RTL Module

```verilog
module adder(
    input  [3:0] a,
    input  [3:0] b,
    output [4:0] sum
);
```

Here:
- `a` and `b` are primary inputs
- `sum` is the primary output

In contrast, a testbench usually does not have primary inputs or outputs because everything required for simulation is generated internally inside the testbench itself.

The testbench is responsible for:
- generating input stimulus
- driving DUT inputs
- generating clock/reset
- monitoring DUT outputs
- checking expected behavior

Therefore, nothing external usually exists outside the testbench during simulation.

### RTL vs Testbench Interaction

```text
Testbench
   ↓
Drives DUT Inputs
   ↓
RTL Design (DUT)
   ↓
Produces Outputs
   ↓
Observed/Checked by Testbench
```

## Synthesizer and Synthesis Flow

Once the RTL design behaves correctly in simulation, the next step is synthesis.

Synthesis is the process of converting RTL code into a gate-level netlist composed of standard cells such as logic gates and flip-flops.

### What is Yosys?

Yosys is an open-source synthesis tool used to synthesize RTL designs written in Verilog.

---

## Basic Synthesis Flow

The synthesis process mainly consists of the following stages:

### 1. Parsing
Reads the RTL code, checks syntax correctness, and builds an internal representation of the design.

Example command:

```tcl
read_verilog design.v
```

---

### 2. Elaboration
Resolves module hierarchy, parameters, and connects instantiated modules.

---

### 3. Optimization
Optimizes the design to improve:
- timing
- area
- power

Optimization techniques may include:
- removing redundant logic
- simplifying Boolean expressions
- restructuring logic paths
- selecting cells with different drive strengths

Design constraints guide the synthesis tool during optimization and timing analysis.

---

### Timing Analysis and PVT Corners

Timing analysis is performed across different PVT (Process, Voltage, Temperature) corners to ensure reliable circuit operation under different manufacturing and environmental conditions.

This is necessary because transistor characteristics and delays (such as RC delays) vary with:
- process variations
- supply voltage
- temperature

These variations directly affect circuit timing and performance.

---

### 4. Technology Mapping
Maps generic RTL logic into technology-specific standard cells available in the target library.

---

### 5. Netlist Generation
Generates the synthesized gate-level netlist.

Example command:

```tcl
write_verilog netlist.v
```

---

## Common Yosys Commands

| Command | Purpose |
|----------|----------|
| `read_verilog` | Reads RTL Verilog files |
| `read_liberty` | Loads standard-cell liberty library |
| `hierarchy` | Resolves module hierarchy |
| `synth` | Runs synthesis |
| `abc` | Performs optimization and technology mapping |
| `write_verilog` | Generates synthesized netlist |

---

## Example Yosys Synthesis Flow

```tcl
read_verilog counter.v

read_liberty -lib skyxyz.lib

hierarchy -top counter

synth -top counter

abc -liberty skyxyz.lib

write_verilog counter_netlist.v
```

---

## RTL vs Gate-Level Netlist Functional Equivalence

After synthesis, the primary inputs and outputs of the design generally remain unchanged.

Because the interface remains the same, the same testbench can usually be reused to verify:
1. RTL design
2. Synthesized gate-level netlist

For the same set of inputs and clock conditions, both should produce matching outputs. This confirms that synthesis preserved the intended RTL functionality.

---

## Simple Understanding of Synthesis

During synthesis, the synthesis tool uses:

- RTL code → to understand functionality
- Liberty library (`.lib`) → to understand available standard cells and timing information
- Constraints → to optimize timing, area, and power

Finally, the synthesis tool generates a technology-mapped gate-level netlist using standard cells from the target library.

---> Practical Tasks Performed <------

- Simulated RTL using Icarus Verilog
- Generated VCD dump file
- Viewed waveforms using GTKWave
- Performed synthesis using Yosys

