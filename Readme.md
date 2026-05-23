# RTL Simulation & Synthesis Basics

# What is RTL?

RTL (Register Transfer Level) design describes what hardware should do on every clock edge.

RTL stands for:

1. **Register** → storage elements such as flip-flops and registers  
2. **Transfer** → movement of data between registers  
3. **Level** → abstraction level used to describe hardware behavior  

In general, RTL acts as a bridge between:
- an algorithmic idea (pseudo code / functional behavior)
- real silicon hardware

In a complete chip design flow, the design passes through multiple stages from specification to final silicon implementation.

The process typically starts with a **Specification (Spec)** that defines:
- required functionality
- performance goals
- power requirements
- area constraints

Based on this specification, engineers write RTL code using HDLs such as:
- Verilog
- VHDL

---

# What is a Specification (Spec)?

A specification is a document that describes:
- what the design should do
- inputs and outputs
- protocols
- timing behavior
- corner cases
- reset behavior
- error handling

---

## Example Specification for a Counter

- 4-bit counter
- increments every clock cycle
- synchronous reset
- wraps from 15 → 0

---

# Need for Testbench and RTL Simulation

## Why RTL Design Needs Verification?

RTL design may violate the specification due to:
1. logic mistakes
2. timing mistakes
3. FSM (Finite State Machine) errors
4. protocol violations

Therefore, RTL must be verified before manufacturing.

This is where:
- simulation
- simulator
- testbench

come into the picture.

A **simulator** is a tool that executes the RTL design along with the testbench and evaluates how signals change over time.

Simulation is used to verify whether the RTL design behaves according to the specification.

---

# What is a Testbench?

A testbench is verification code written specifically to test and validate the DUT (Design Under Test).

It is usually written in:
- Verilog
- SystemVerilog

A testbench is used only for simulation purposes and is not synthesized into hardware.
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
# Understanding Important Icarus Verilog Commands

## 1. Compile RTL + Testbench

```bash
iverilog good_mux.v tb_good_mux.v
```

### What this command does:
- Reads RTL and testbench files
- Checks syntax correctness
- Elaborates module hierarchy
- Creates simulation executable

By default, Icarus Verilog generates an executable file named:

```text
a.out
```

---

## What is `vvp`?

`vvp` is the runtime engine (simulator backend) used by Icarus Verilog to execute the compiled simulation executable.

Relationship between `iverilog` and `vvp`:

```text
iverilog
    ↓
Compiles Verilog code into simulation executable

vvp
    ↓
Executes the compiled simulation executable
```

---

## 2. Run Simulation

In many systems, the generated executable can be run directly using:

```bash
./a.out
```

Internally, this executable is associated with the `vvp` runtime.

The equivalent explicit command is:

```bash
vvp a.out
```

Both commands perform the same simulation execution.

---

### What happens during simulation:
- Testbench starts executing
- Inputs are applied to DUT
- RTL logic gets evaluated
- Outputs change over time
- Waveform dump file gets generated

Generated waveform file:

```text
waveform.vcd
```

---

## 3. Open Waveform in GTKWave

```bash
gtkwave waveform.vcd
```

### Purpose:
- Displays signal transitions graphically
- Helps analyze RTL behavior over time

---

# Complete Icarus Verilog Simulation Flow

```text
RTL + Testbench
        ↓
     iverilog
(Compile + Elaborate)
        ↓
 Creates Executable
      (a.out)
        ↓
 ./a.out  or  vvp a.out
   (Run Simulation)
        ↓
 Generates waveform.vcd
        ↓
      GTKWave
(View Waveforms)
```

---

# Commonly Used Icarus Verilog Commands

| Command | Purpose |
|----------|----------|
| `iverilog file.v` | Compile Verilog code |
| `iverilog rtl.v tb.v` | Compile RTL + testbench |
| `iverilog -o sim rtl.v tb.v` | Create custom executable name |
| `./a.out` | Run simulation executable |
| `vvp a.out` | Run simulation using vvp runtime |
| `gtkwave waveform.vcd` | Open waveform viewer |

---

# Example Using Custom Executable Name

## Compile

```bash
iverilog -o mux_sim good_mux.v tb_good_mux.v
```

This creates executable:

```text
mux_sim
```

---

## Run Simulation

```bash
./mux_sim
```

OR

```bash
vvp mux_sim
```

---

## View Waveform

```bash
gtkwave waveform.vcd
```

---

# Important System Tasks Used in Testbench

| System Task | Purpose |
|--------------|----------|
| `$dumpfile` | Creates VCD waveform file |
| `$dumpvars` | Dumps signal values into VCD |
| `$finish` | Ends simulation |

---

# Simple Understanding

```text
iverilog
    ↓
Compiles RTL + Testbench

./a.out or vvp a.out
    ↓
Runs Simulation

GTKWave
    ↓
Displays Waveforms
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

# Important Point While Running GLS with Multiple Modules

If the top-level design instantiates other modules internally, all dependent Verilog module files must also be provided during compilation.

For example:

```verilog
module ripple_carry_adder(
    input  [3:0] a,
    input  [3:0] b,
    output [3:0] sum
);

adder u1 (...);
adder u2 (...);

endmodule
```

Here:
- `ripple_carry_adder.v` is the top module
- `adder.v` contains the definition of module `adder`

Since `ripple_carry_adder.v` internally uses (`instantiates`) the `adder` module, the simulator must know the definition of `adder`.

Therefore, while compiling, all dependent module files must be included.

---

# Example Compilation Command

```bash
iverilog ripple_carry_adder_netlist.v rca.v adder.v tb_rca.v
```

Here:
- `ripple_carry_adder_netlist.v` → synthesized netlist
- `rca.v` → RTL/top-level module
- `adder.v` → lower-level instantiated module
- `tb_rca.v` → testbench

---

# Why This is Necessary?

During elaboration, the simulator resolves:
- module hierarchy
- instantiated modules
- interconnections

If any instantiated module definition is missing, the simulator throws errors such as:

```text
Unknown module type
Module not found
```

---

# Simple Understanding

```text
Top Module
    ↓
Instantiates Submodules
    ↓
Simulator Needs All Module Definitions
    ↓
All Related .v Files Must Be Passed
```

---

# Important Note

This applies to:
- RTL simulation
- Gate-Level Simulation (GLS)

Whenever a design hierarchy contains multiple modules, all required Verilog files must be included during compilation.

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

