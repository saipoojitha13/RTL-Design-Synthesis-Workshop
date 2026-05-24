
#  RTL Simulation & Synthesis Basics 🧠

## Repository Contents

- [1. Spec & RTL Understanding](#1-spec--rtl-understanding)
- [2. Need for Testbench and RTL Simulation](#2-need-for-testbench-and-rtl-simulation)
- [3. Synthesizer and Synthesis Flow](#3-synthesizer-and-synthesis-flow)
- [4. RTL vs Gate-Level Netlist Functional Equivalence](#4-rtl-vs-gate-level-netlist-functional-equivalence)
- [5. Liberty File, Cell Variants, Slack & Cell Selection](#5-liberty-file-cell-variants-slack--cell-selection)
- [6. Synthesis vs Simulation Mismatch](#6-synthesis-vs-simulation-mismatch)
- [7. Blocking vs Non-Blocking Assignments](#7-blocking-vs-non-blocking-assignments)
- [8. Optimization techniques](#8-optimization-techniques)
- [9. Basics on RTL coding styles](#9-basics-on-rtl-coding-styles)
- [10. Final Understanding](#10-final-understanding)


# 1. Spec & RTL understanding

## 📌 What is RTL?

RTL (Register Transfer Level) design describes what hardware should do on every clock edge.

RTL stands for:

1. **Register** → storage elements such as flip-flops and registers  
2. **Transfer** → movement of data between registers  
3. **Level** → abstraction level used to describe hardware behavior  

- In general, RTL acts as a bridge between an algorithmic idea (pseudo code / functional behavior) & real silicon hardware

- In a complete chip design flow, the design passes through multiple stages from specification to final silicon implementation.

🔄 The process typically starts with a **Specification (Spec)** that defines:
- required functionality
- performance goals
- power requirements
- area constraints

Based on this specification, engineers write RTL code using HDLs such as:
- Verilog
- VHDL

---

## What is a Specification (Spec)?

A specification is a document that describes:
- what the design should do
- inputs and outputs
- protocols
- timing behavior
- corner cases
- reset behavior
- error handling

---
### 🧠 Simple Example Specification : 4-Bit Counter

### Functional Requirements
- Counter width = 4 bits
- Counter increments on every positive clock edge
- Counter wraps from `15 → 0`
- Counter supports synchronous reset

### Inputs
- `clk` → clock signal
- `rst` → synchronous reset

### Outputs
- `count[3:0]` → 4-bit counter output

### Reset Behavior
- When `rst = 1`, counter resets to `0`

### Timing Behavior
- Counter updates on positive edge of clock

[⬆ Back to Repository Contents](#repository-contents)

# 2. Need for Testbench and RTL Simulation

### Why RTL Design Needs Verification?

RTL design may violate the specification due to:
1. logic mistakes
2. timing mistakes
3. FSM (Finite State Machine) errors
4. protocol violations

- Therefore, RTL must be verified before manufacturing. This is where simulation/simulator/testbench come into the picture.

A **simulator** is a tool that executes the RTL design along with the testbench and evaluates how signals change over time.

Simulation is used to verify whether the RTL design behaves according to the specification or not.

### What is a Testbench?

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


###  📌 Points to Observe While Dealing with RTL Code and Testbench

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
### Understanding Important Icarus Verilog Commands

- 1. Compile RTL + Testbench

```bash
iverilog good_mux.v tb_good_mux.v
```

- What this command does:
- Reads RTL and testbench files
- Checks syntax correctness
- Elaborates module hierarchy
- Creates simulation executable

By default, Icarus Verilog generates an executable file named:

```text
./a.out
```

---

- here we need to know What is `vvp`?

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

- 2. Run Simulation

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

- What happens during simulation:
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

- 3. Open Waveform in GTKWave

```bash
gtkwave waveform.vcd
```

- Purpose:
- Displays signal transitions graphically
- Helps analyze RTL behavior over time

---

- Complete Icarus Verilog Simulation Flow

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

### Commonly Used Icarus Verilog Commands

| Command | Purpose |
|----------|----------|
| `iverilog file.v` | Compile Verilog code |
| `iverilog rtl.v tb.v` | Compile RTL + testbench |
| `iverilog -o sim rtl.v tb.v` | Create custom executable name |
| `./a.out` | Run simulation executable |
| `vvp a.out` | Run simulation using vvp runtime |
| `gtkwave waveform.vcd` | Open waveform viewer |

---

### Example Using Custom Executable Name

- Compile

```bash
iverilog -o mux_sim good_mux.v tb_good_mux.v
```

This creates executable:

```text
mux_sim
```

---

#### Run Simulation

```bash
./mux_sim
```

OR

```bash
vvp mux_sim
```

---

#### View Waveform

```bash
gtkwave waveform.vcd
```

---

### Important System Tasks Used in Testbench

| System Task | Purpose |
|--------------|----------|
| `$dumpfile` | Creates VCD waveform file |
| `$dumpvars` | Dumps signal values into VCD |
| `$finish` | Ends simulation |

---

### Simple Understanding

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
[⬆ Back to Repository Contents](#repository-contents)

# 3. Synthesizer and Synthesis Flow

Once the RTL design behaves correctly in simulation, the next step is synthesis.

Synthesis is the process of converting RTL code into a gate-level netlist composed of standard cells such as logic gates and flip-flops.

### Basic Synthesis Flow

The synthesis process mainly consists of the following stages:

 1. Parsing
Reads the RTL code, checks syntax correctness, and builds an internal representation of the design.

Example command:

```tcl
read_verilog design.v
```

---

 2. Elaboration
Resolves module hierarchy, parameters, and connects instantiated modules.

---

 3. Optimization
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

 4. Technology Mapping
Maps generic RTL logic into technology-specific standard cells available in the target library.

---

 5. Netlist Generation
Generates the synthesized gate-level netlist.

Example command:

```tcl
write_verilog netlist.v
```

---

### what is yosys ? Common Yosys Commands

- Yosys is an open-source synthesis tool used to synthesize RTL designs written in Verilog.

| Command | Purpose |
|----------|----------|
| `read_verilog` | Reads RTL Verilog files |
| `read_liberty` | Loads standard-cell liberty library |
| `hierarchy` | Resolves module hierarchy |
| `synth` | Runs synthesis |
| `abc` | Performs optimization and technology mapping |
| `write_verilog` | Generates synthesized netlist |

---

### Example of Yosys Synthesis Flow

```tcl
read_verilog counter.v

read_liberty -lib skyxyz.lib

hierarchy -top counter

synth -top counter

abc -liberty skyxyz.lib

write_verilog counter_netlist.v
```

---

### Simple Understanding of Synthesis and some special commands

During synthesis, the synthesis tool uses:

- RTL code → to understand functionality
- Liberty library (`.lib`) → to understand available standard cells and timing information
- Constraints → to optimize timing, area, and power

Finally, the synthesis tool generates a technology-mapped gate-level netlist using standard cells from the target library.

### Some Special Synth Commands

- 1. `synth` command

`synth` is a high-level synthesis command in Yosys.

It performs multiple synthesis stages automatically, such as:
- hierarchy checking
- process conversion
- optimization
- FSM optimization
- mux optimization
- generic logic generation

In simple terms:

```text
synth converts RTL into generic synthesized logic
```

---

- 🧠 Internally `synth` Performs

Some internal steps include:
- parsing RTL
- elaboration
- converting always blocks
- inferring muxes
- inferring flip-flops
- optimization passes

---

- 📌 Example

RTL:

```verilog
always @(posedge clk)
    q <= d;
```

After `synth`, Yosys may internally create:

```text
$_DFF_P_
```

This is:
- generic flip-flop representation
- NOT actual SKY130 cell yet

Similarly:

```text
$_AND_
$_OR_
$_MUX_
```

may also be created.

---

- 📌 Important Point

After `synth`:
- logic is synthesized
- but not fully mapped into actual technology cells yet

Technology mapping happens later using:
- `dfflibmap`
- `abc`

---

- 2. `dfflibmap Command`

- 📌 Syntax

```tcl
dfflibmap -liberty skyxyz.lib
```

Example:

```tcl
dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
```

---

- 📌 What Does `dfflibmap` Do?

`dfflibmap` maps:
- generic flip-flops
- generic latches

into:
- actual technology-specific sequential cells

from the liberty library.

---

- 🧠 Why is it Needed?

After `synth`, Yosys creates generic sequential elements like:

```text
$_DFF_P_
```

But real ASIC hardware needs actual cells like:

```text
sky130_fd_sc_hd__dfxtp_1
```

So:

```text
dfflibmap converts generic DFFs
into real library DFF cells
```

---

- 📌 Example

Before `dfflibmap`:

```text
$_DFF_P_
```

After `dfflibmap`:

```text
sky130_fd_sc_hd__dfxtp_1
```

Now synthesized netlist contains:
- actual library flip-flops

---

- 📌 Important Point

`dfflibmap` handles:
- sequential logic only

Example:
- D flip-flops
- latches

It does NOT map:
- AND gates
- OR gates
- muxes

Those are handled later by:
- `abc`

---

- 3. opt_clean -purge Command

- 📌 Syntax

```tcl
opt_clean -purge
```

---

- 📌 What Does `opt_clean` Do?

`opt_clean` removes:
- unused logic
- unused wires
- redundant cells

from synthesized design.

In simple terms:

```text
Cleanup unnecessary hardware
```

---

- 📌 What Does `-purge` Mean?

The `-purge` option performs:
- more aggressive cleanup

It removes:
- completely unused signals
- temporary/internal wires
- dangling logic

---

- Example

Suppose RTL contains:

```verilog
wire temp;

assign temp = a & b;
```

But:

```text
temp is never used anywhere
```

Then:

```tcl
opt_clean -purge
```

can remove:
- temp wire
- associated unused gate

This reduces:
- area
- unnecessary logic

---

- 📌 Why is Cleanup Important?

During synthesis:
- many temporary signals
- intermediate logic
- unused wires

may get generated.

Cleanup improves:
- netlist readability
- optimization quality
- area efficiency

---

- Summary of Commands

| Command | Purpose |
|---|---|
| `synth -top top_module` | Performs generic synthesis |
| `dfflibmap -liberty file.lib` | Maps generic DFFs/latches to library cells |
| `opt_clean -purge` | Removes unused logic and wires |

---

### 🚀 Simple Complete Flow

```tcl
read_verilog design.v

read_liberty -lib sky130.lib

synth -top top

dfflibmap -liberty sky130.lib

abc -liberty sky130.lib

opt_clean -purge

write_verilog design_netlist.v
```
[⬆ Back to Repository Contents](#repository-contents)

# 4. RTL vs Gate-Level Netlist Functional Equivalence

After synthesis, the RTL design is converted into a gate-level netlist composed of technology-specific standard cells such as:
- AND gates
- OR gates
- multiplexers
- flip-flops

Although the internal implementation changes during synthesis, the external interface of the design usually remains unchanged.

This means:
- primary inputs remain the same
- primary outputs remain the same
- functionality should remain the same

Because the interface remains unchanged, the same testbench can generally be reused to verify both:
1. RTL design
2. Synthesized gate-level netlist

For the same set of:
- inputs
- clock conditions
- reset conditions

both RTL simulation and Gate-Level Simulation (GLS) should produce matching outputs.

This confirms that the synthesis process preserved the intended RTL functionality correctly.


### What is Gate-Level Simulation (GLS)?

Gate-Level Simulation (GLS) is the process of simulating the synthesized gate-level netlist using a testbench.

GLS helps verify:
- synthesized hardware behavior
- functional equivalence with RTL
- synthesis correctness
- timing-related issues
- initialization/reset behavior

---

- RTL Simulation vs GLS

| RTL Simulation | Gate-Level Simulation (GLS) |
|----------------|-----------------------------|
| Simulates RTL behavioral code | Simulates synthesized gate-level netlist |
| Faster simulation | Slower simulation |
| Uses RTL operators | Uses standard cells/gates |
| Easier debugging | Closer to real hardware |

---



### understanding the Generation of Gate-Level Netlist & GLS Using Yosys with one simple example 

- Example : RTL Design (good_mux.v)

```verilog
module good_mux(
    input  i0,
    input  i1,
    input  sel,
    output reg y
);

always @(*) begin
    if(sel)
        y = i1;
    else
        y = i0;
end

endmodule
```

---

- Testbench (tb_good_mux.v)

```verilog
module tb_good_mux;

reg i0, i1, sel;
wire y;

good_mux uut (
    .i0(i0),
    .i1(i1),
    .sel(sel),
    .y(y)
);

initial begin
    $dumpfile("waveform.vcd");
    $dumpvars(0, tb_good_mux);

    i0 = 0; i1 = 1; sel = 0;
    #10;

    sel = 1;
    #10;

    i0 = 1; i1 = 0; sel = 0;
    #10;

    sel = 1;
    #10;

    $finish;
end

endmodule
```


- step 1: performing simulation on RTL

```bash
iverilog good_mux.v tb_good_mux.v
./a.out
Generating tb_good_mux.vcd
```
open the dump using gtkwave and analyze the waveform results 

---

- Step 2 : Run Synthesis Commands

```tcl
yosys

read_liberty -lib skyxyz.lib 

read_verilog good_mux.v

synth -top good_mux

abc -liberty skyxyz.lib 

write_verilog good_mux_netlist.v
```

This generates synthesized gate-level netlist:

```text
good_mux_netlist.v
```

---

- step 3 : Compile Gate-Level Netlist + Testbench

```bash
iverilog good_mux_netlist.v tb_good_mux.v
```

---

- Run Simulation

```bash
./a.out
```

- step 4 : Open Waveform of GLS simulation dump and analyze the results by comparing with the previous rtl simulation dump

```bash
gtkwave waveform.vcd
```
- Note: With the following above 5 steps, we can understand, in design we are verifying have any synthesis-simulation mismatch or not.
---

### a simple understanding of Complete GLS Flow

```text
RTL Design
     ↓
Synthesis using Yosys
     ↓
Gate-Level Netlist
(good_mux_netlist.v)
     ↓
GLS using Icarus Verilog
     ↓
Waveform Generation
     ↓
GTKWave Analysis and cross-comparing the results of RTL simulation waveform dump
```

---

### Simple Understanding

```text
RTL Simulation
    ↓
Checks RTL functionality

GLS
    ↓
Checks synthesized hardware functionality
```

Both outputs should match for the same testbench inputs.

### Important Point While Running GLS with Multiple Modules

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

- Example Compilation Command

```bash
iverilog ripple_carry_adder_netlist.v rca.v adder.v tb_rca.v
```

Here:
- `ripple_carry_adder_netlist.v` → synthesized netlist
- `rca.v` → RTL/top-level module
- `adder.v` → lower-level instantiated module
- `tb_rca.v` → testbench

---

- Why This is Necessary?

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

- Simple Understanding

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

- Important Note

This applies to:
- RTL simulation
- Gate-Level Simulation (GLS)

Whenever a design hierarchy contains multiple modules, all required Verilog files must be included during compilation.

[⬆ Back to Repository Contents](#repository-contents)

# 5. Liberty File, Cell Variants, Slack & Cell Selection

 - **Liberty file (`.lib`)** is a standard-cell timing library file used during synthesis and timing analysis.

It contains detailed information about:
- standard cells
- timing delays
- power consumption
- drive strengths
- operating conditions (PVT corners)

The synthesis and STA (Static Timing Analysis) tools use this file to understand how real hardware cells behave.

---

### Why Liberty File is Needed?

During synthesis, the RTL design only describes functionality.

The synthesis tool still needs information about:
- available hardware cells
- timing delay of each cell
- power consumption
- drive strength
- setup and hold timing

This information is provided by the Liberty (`.lib`) file.

Without a liberty file:
- synthesis cannot map RTL into actual standard cells
- timing analysis cannot be performed

---

- Simple Understanding

```text
RTL Design
     ↓
Synthesis Tool
     ↓
Uses Liberty File (.lib)
     ↓
Chooses Actual Standard Cells
     ↓
Generates Gate-Level Netlist
```

---

### Example Liberty File Name

```text
sky130_fd_sc_hd__tt_025C_1v80.lib
```

This file name itself contains important technology and PVT information.

---

- Breaking Down the Liberty File Name

 1. sky130

```text
sky130
```

Represents:
- SKY130 technology node
- approximately 130nm process technology

Feature size is roughly:

```text
0.13 µm
```

---

 2. fd


Means:


```text
Foundry Device
```

It indicates the library is provided by the foundry.

---

 3. sc

Means:

```text
Standard Cells
```

These are pre-designed digital logic cells such as:
- AND gates
- OR gates
- multiplexers
- flip-flops
- buffers

---

 4. hd


Means:

```text
High Density Library
```

This library is optimized for:
- smaller area
- higher cell density

Different library flavors may exist such as:
- `hd` → High Density
- `hs` → High Speed
- `lp` → Low Power

Each library flavor has different tradeoffs between:
- speed
- power
- area

---

 5. PVT Corners in Liberty Files

PVT stands for:

| Parameter | Meaning |
|------------|----------|
| P | Process |
| V | Voltage |
| T | Temperature |

Timing behavior changes depending on these conditions.

---

- Process (P)

Process variation occurs due to manufacturing differences in transistors.

Examples:
- Fast transistors
- Slow transistors

Common process corners:

| Corner | Meaning |
|--------|----------|
| TT | Typical NMOS + Typical PMOS |
| FF | Fast NMOS + Fast PMOS |
| SS | Slow NMOS + Slow PMOS |
| FS | Fast NMOS + Slow PMOS |
| SF | Slow NMOS + Fast PMOS |

---

- Voltage (V)

Timing changes with supply voltage.

Higher voltage:
- faster switching
- lower delay

Lower voltage:
- slower switching
- higher delay

Example:

```text
1v80 = 1.80V supply
```

---

- Temperature (T)

Temperature affects transistor performance.

Higher temperature generally:
- increases delay
- slows transistor switching

Example:

```text
025C = 25°C
```

---

- Understanding Example Completely

```text
sky130_fd_sc_hd__tt_025C_1v80.lib
```

| Part | Meaning |
|------|----------|
| sky130 | SKY130 technology (130nm) |
| fd | Foundry provided |
| sc | Standard cell library |
| hd | High density library |
| tt | Typical process corner |
| 025C | 25°C temperature |
| 1v80 | 1.80V supply voltage |

---

### How Liberty File is Used in Yosys

Example command:

```tcl
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
```

Purpose:
- loads standard-cell timing library
- provides timing and cell information
- helps technology mapping
- enables timing-aware synthesis

---

- Important Information Present Inside Liberty File

A liberty file contains:
- cell names
- timing delays
- setup time
- hold time
- power data
- pin capacitance
- drive strength
- transition delays
- operating conditions

---

- Simple Final Understanding

```text
RTL
   ↓
Synthesis Tool
   ↓
Reads Liberty File (.lib)
   ↓
Understands Available Standard Cells
   ↓
Maps RTL into Real Hardware Cells
   ↓
Generates Gate-Level Netlist
```

The liberty file acts like a hardware behavior database for the synthesis and timing tools.


### Standard Cell Libraries and Different Cell Variants

A liberty file (`.lib`) does not contain only one version of a logic gate.

Instead, it contains multiple versions of the same logic function with:
- different drive strengths
- different transistor sizes
- different speed, area, and power tradeoffs

Example:

```text
sky130_fd_sc_hd__and2_0
sky130_fd_sc_hd__and2_1
sky130_fd_sc_hd__and2_2
sky130_fd_sc_hd__and2_4
```

All of these are:
- 2-input AND gates
- perform the same Boolean logic function

But they differ in:
- transistor size
- drive capability
- delay
- area
- power consumption

---

- Note: General Standard Cell Naming Format

```text
<library>__<function>_<drive_strength>
```

Example:

```text
sky130_fd_sc_hd__and2_2
```

Breaking it down:

| Part | Meaning |
|------|----------|
| sky130_fd_sc_hd | Library name |
| and2 | 2-input AND gate |
| 2 | Drive strength |

---

Note:  What is Drive Strength?

Drive strength indicates how much current a gate can source or sink to charge/discharge load capacitance.

A stronger cell:
- can drive larger loads
- switches signals faster
- has lower delay

But:
- consumes more area
- consumes more power

---

- Why Larger Cells are Faster?

The relation is:

:contentReference[oaicite:0]{index=0}

From this equation:

- Larger current (`I`) changes voltage faster
- Faster voltage transition means lower delay

Larger cells contain:
- bigger transistors
- higher current-driving capability

Therefore:
- larger cells charge/discharge capacitance faster
- signals propagate faster

---

- Important Point

Drive strength does **not** change the logic functionality.

For example:

```text
and2_0
and2_2
and2_4
```

All perform:

```text
Y = A & B
```

Only their electrical capability changes.

---

- Comparison of Different Drive Strength Cells

| Property | and2_0 | and2_2 | and2_4 |
|----------|---------|---------|---------|
| Logic Function | Same | Same | Same |
| Transistor Size | Small | Medium | Large |
| Drive Current | Low | Medium | High |
| Delay | High | Medium | Low |
| Area | Small | Medium | Large |
| Power Consumption | Low | Medium | High |
| Best Use Case | Small loads | Moderate loads | Heavy loads / Timing-critical paths |

---

### Fast Cells vs Slow Cells

Liberty libraries contain different types of cells optimized for different purposes.

---

- Fast Cells

Fast cells usually:
- use larger transistors
- provide higher drive current
- reduce delay

Advantages:
- faster signal propagation
- better timing performance

Disadvantages:
- larger area
- higher power consumption

Used in:
- timing-critical paths
- high-speed designs

---

- Slow Cells

Slow cells usually:
- use smaller transistors
- provide lower drive current
- have higher delay

Advantages:
- lower power consumption
- smaller area

Disadvantages:
- slower switching speed

Used in:
- non-critical timing paths
- low-power designs

---

- How Synthesis Tool Chooses Cells

During synthesis and optimization:
- the tool analyzes timing constraints
- checks load capacitance
- checks timing slack

Then it automatically selects suitable cells.

Example:
- critical path → faster/larger cells
- non-critical path → smaller/slower cells

This process is called:

```text
Cell Sizing / Drive Strength Optimization
```

---

- Simple Understanding

```text
Same Logic Function
        ↓
Multiple Cell Variants Exist
        ↓
Different Drive Strengths
        ↓
Different Speed / Area / Power Tradeoffs
```

---

- Final Understanding

A liberty file contains many versions of the same logic gates with different:
- drive strengths
- transistor sizes
- timing characteristics
- power characteristics

This allows synthesis tools to optimize the design for:
- timing
- area
- power

depending on the design constraints and timing requirements.

### What is Slack in Timing Analysis?

Slack indicates whether a timing path meets or violates timing requirements.

It tells us:

```text
How much timing margin is available
```

OR

```text
Whether data arrived too late or too early
```

Slack is one of the most important concepts in:
- synthesis
- Static Timing Analysis (STA)
- timing closure

---

- Simple Timing Path

```text
Launch Flip-Flop
        ↓
Combinational Logic
        ↓
Capture Flip-Flop
```

Data:
- launches from first flip-flop
- travels through combinational logic
- must reach destination flip-flop within required timing

---

- Setup Timing Slack

For setup timing:

```text
Slack = Required Time - Arrival Time
```

Where:
- Required Time → latest allowed arrival time
- Arrival Time → actual data arrival time

---

- Positive Slack

If:

```text
Slack > 0
```

then:

```text
Data arrived before deadline
```

Meaning:
- timing is safe
- no setup violation
- path has timing margin

Example:

```text
Required Time = 10ns
Arrival Time  = 8ns

Slack = 10 - 8 = +2ns
```

This means:
- data arrived 2ns early
- design is timing-safe

---

- Zero Slack

If:

```text
Slack = 0
```

then:

```text
Data arrived exactly at required time
```

Meaning:
- timing just meets requirement
- no safety margin exists

Example:

```text
Required Time = 10ns
Arrival Time  = 10ns

Slack = 0ns
```

---

- Negative Slack

If:

```text
Slack < 0
```

then:

```text
Data arrived too late
```

Meaning:
- setup timing violation occurs
- circuit may fail at target frequency

Example:

```text
Required Time = 10ns
Arrival Time  = 12ns

Slack = 10 - 12 = -2ns
```

Meaning:
- data missed deadline by 2ns
- setup violation exists

---

- Simple Understanding

```text
Positive Slack
    ↓
Timing Safe

Zero Slack
    ↓
Just Meets Timing

Negative Slack
    ↓
Timing Violation
```

---

### Why Slack is Important for Cell Selection?

During synthesis and timing optimization, the tool checks slack values on timing paths.

Depending on slack:
- it may choose faster cells
- or smaller/slower cells

---

- Case 1 : Negative Slack (Timing Violation)

Suppose:

```text
Slack = -1ns
```

Meaning:
- path is too slow
- data arrives late

To fix this, synthesis tool may:
- use larger drive-strength cells
- use faster cells
- reduce combinational delay

Example:

```text
and2_0  → replaced by → and2_4
```

Why?

Because:
- larger cells provide more current
- charge/discharge capacitance faster
- reduce delay

---

- Case 2 : Large Positive Slack

Suppose:

```text
Slack = +5ns
```

Meaning:
- path is much faster than required
- extra timing margin exists

Tool may optimize for:
- lower area
- lower power

So it may replace:

```text
and2_4 → and2_1
```

Why?

Because:
- smaller cells consume less power
- smaller area
- timing is still safe

---

- Relation Between Slack and Cell Size

| Slack Condition | Tool Action |
|----------------|-------------|
| Negative Slack | Use faster/larger cells |
| Small Positive Slack | Keep optimized cells |
| Large Positive Slack | Use smaller/slower cells for power saving |

---

- How Hold Timing is Different?

Setup timing checks:

```text
Data should not arrive too late
```

Hold timing checks:

```text
Data should not arrive too early
```

---

- Hold Slack

For hold timing:

```text
Hold Slack = Arrival Time - Required Hold Time
```

---

- Hold Violation

If data changes too quickly after clock edge:

```text
Hold Slack < 0
```

then:
- destination flip-flop may capture wrong data
- hold violation occurs

---

- Important Point

Large fast cells can sometimes create hold violations.

Why?

Because:
- faster cells reduce delay too much
- data reaches destination too early

---

- Example

Suppose tool aggressively replaces:

```text
and2_1 → and2_8
```

Now:
- path becomes extremely fast
- setup timing improves
- but data may arrive too early

This can create:

```text
Hold Violation
```

---

- How Tool Fixes Hold Violations?

To fix hold violations, tools may:
- insert buffers
- use smaller/slower cells
- increase path delay intentionally

Example:

```text
and2_8 → and2_2
```

OR

```text
Insert Buffer
```

---

#### Important Tradeoff

```text
Faster Cells
    ↓
Improve Setup Timing
But
May Hurt Hold Timing

Slower Cells
    ↓
Help Hold Timing
But
May Hurt Setup Timing
```

---

#### Real Synthesis Optimization Goal

The synthesis and STA tools try to achieve:

```text
No Setup Violations
AND
No Hold Violations
```

while also optimizing:
- area
- power
- performance

---

#### Final Understanding

Slack tells the synthesis and timing tools whether a path is:
- too slow
- too fast
- or timing-safe

Based on slack analysis, the tool automatically selects:
- fast cells
- slow cells
- larger drive strengths
- smaller drive strengths

to balance:
- setup timing
- hold timing
- power
- area
- performance

[⬆ Back to Repository Contents](#repository-contents)

# 6. Synthesis vs Simulation Mismatch

### 📌 What is Synthesis vs Simulation Mismatch?

A synthesis vs simulation mismatch happens when:

```text
RTL Simulation Output
            ≠
Synthesized Hardware Output
```

- This means the RTL code behaves one way during simulation but after synthesis, the actual hardware behaves differently

In simple terms:

```text
Simulation says design is correct
But synthesized hardware behaves incorrectly
```

This is dangerous because:
- simulation may pass successfully
- but FPGA/ASIC hardware may fail

---

### 🧠 Why Does This mismatch Happen?

Simulation and synthesis tools do not interpret bad RTL code exactly the same way.

- Simulator focuses on:
  - executing RTL behavior
  - following event changes
  - software-like execution

- Synthesis tool focuses on:
  - converting RTL into actual hardware gates
  - creating real combinational/sequential circuits

- So poorly written RTL, incomplete coding styles and ambiguous logic can produce different results in simulation and synthesized hardware.

- 📌 Common Reasons for Mismatch

Some common causes are:

1. Incomplete sensitivity list
2. Blocking vs non-blocking misuse
3. Missing ELSE or DEFAULT statements
4. Uninitialized signals
5. Improper latch inference
6. Delays (`#5`) used in RTL
7. X/Z handling differences

---

- Example : Incomplete Sensitivity List

This is one of the most famous synthesis-simulation mismatch examples.

---

- ❌ Bad RTL Code

```verilog
module bad_mux(
    input  i0,
    input  i1,
    input  sel,
    output reg y
);

always @(sel) begin
    if(sel)
        y = i1;
    else
        y = i0;
end

endmodule
```

---

- 📌 What is Wrong Here?

Sensitivity list contains only:

```verilog
@(sel)
```

But inside logic we are also using:
- `i0`
- `i1`

They are missing from sensitivity list.

---

- How Simulator Behaves?

Simulator executes the `always` block only when:

```text
sel changes
```

Suppose:

```text
sel = 0
i0 = 0
```

Output:

```text
y = 0
```

Now if:

```text
i0 changes from 0 → 1
```

BUT:

```text
sel did not change
```

So simulator:
- does NOT execute always block
- output y remains old value

Simulation output becomes:

```text
Incorrect/Stale Output
```

---

- How Synthesis Tool Behaves?

Synthesis tool analyzes RTL logic mathematically.

It understands:

```text
This is actually a combinational MUX
```

So synthesized hardware becomes:

```text
y = sel ? i1 : i0
```

Real hardware output changes immediately whenever:
- `i0`
- `i1`
- `sel`

changes.

So synthesized hardware behaves correctly.

---

- Final Result

```text
Simulation Output
        ≠
Synthesized Hardware Output
```

This becomes a:
- synthesis vs simulation mismatch

---

- Understanding with Example

Suppose:

| sel | i0 | i1 | Expected y |
|----|----|----|----|
| 0 | 0 | X | 0 |

Now:

```text
i0 changes 0 → 1
```

---

- RTL Simulation

Simulator checks:

```text
Did sel change?
```

Answer:

```text
No
```

So:
- always block not executed
- y remains 0

Simulation result:

```text
y = 0 ❌
```

---

- Synthesized Hardware

Actual hardware continuously responds to all inputs.

So:

```text
y immediately becomes 1
```

Hardware result:

```text
y = 1 ✅
```

---

- ✅ Correct RTL Coding Style

Use:

```verilog
always @(*)
```

OR

```systemverilog
always_comb
```

---

- ✅ Correct Version

```verilog
module good_mux(
    input  i0,
    input  i1,
    input  sel,
    output reg y
);

always @(*) begin
    if(sel)
        y = i1;
    else
        y = i0;
end

endmodule
```

---

- 📌 Why `@(*)` is Important?

`@(*)` automatically includes:
- all input signals
- all RHS signals used inside always block

So simulator correctly reevaluates logic whenever:
- `i0`
- `i1`
- `sel`

changes.

Now:
- simulation matches hardware
- no mismatch occurs

---

- 🧠 Simple Understanding

- Bad RTL

```text
Incomplete sensitivity list
        ↓
Simulator misses some signal changes
        ↓
Wrong simulation behavior
```

---

- Synthesized Hardware

```text
Hardware still becomes proper combinational circuit
        ↓
Hardware output behaves differently
```

---

- 📌 Important Lesson

Simulation correctness does NOT guarantee:
- synthesis correctness
- hardware correctness

Good RTL coding practices are extremely important to ensure:

```text
RTL Simulation Output
            =
Synthesized Hardware Output
```

---

- ✅ Best Practices to Avoid Mismatch

- Use `always @(*)` for combinational logic
- Use non-blocking (`<=`) for sequential logic
- Avoid incomplete IF/CASE statements
- Initialize signals properly
- Avoid delays (`#`) in synthesizable RTL
- Perform Gate-Level Simulation (GLS)

---

[⬆ Back to Repository Contents](#repository-contents)

# 7. Blocking vs Non-Blocking Assignments

Understanding blocking and non-blocking assignments is very important in Verilog because improper usage can create:

- synthesis vs simulation mismatches
- incorrect sequential behavior
- race conditions
- unexpected hardware behavior

---

- 📌 Two Types of Assignments in Verilog

| Type | Symbol |
|------|--------|
| Blocking Assignment | `=` |
| Non-Blocking Assignment | `<=` |

---

- 🧠 Simple Understanding

#### Blocking Assignment (`=`)

```text
Executes statements one-by-one in sequence
```

Like:

```text
First line finishes
THEN
next line executes
```

Similar to:
- normal software programming

---

#### Non-Blocking Assignment (`<=`)

```text
All assignments happen in parallel
```

Values are updated:
- together
- at end of current time step

This behavior matches:
- real flip-flop hardware

---

- 📌 Simple Analogy

- Blocking (`=`)

Imagine:

```text
Person A gives book to B
THEN
B gives book to C
```

Operations happen:
- one after another

---

- Non-Blocking (`<=`)

Imagine:

```text
Everyone updates simultaneously on clock edge
```

This is how:
- real registers/flip-flops behave in hardware

---

- Example 1 : Blocking Assignment Problem

#### ❌ Wrong Sequential Logic

```verilog
module blocking_example(
    input clk,
    input d,
    output reg q1,
    output reg q2
);

always @(posedge clk) begin
    q1 = d;
    q2 = q1;
end

endmodule
```

---

- 🧠 What Simulator Does?

Suppose before clock edge:

```text
d  = 1
q1 = 0
q2 = 0
```

At clock edge:

---

- Step 1

```verilog
q1 = d;
```

Immediately:

```text
q1 becomes 1
```

---

- Step 2

```verilog
q2 = q1;
```

Now simulator uses:
- updated q1 value

So:

```text
q2 also becomes 1
```

---

- 📌 Final Simulation Result

After one clock edge:

```text
q1 = 1
q2 = 1
```

---

- ⚠️ But Real Hardware Works Differently

In actual hardware:
- both flip-flops sample inputs simultaneously at clock edge

So:

```text
q2 should capture OLD q1 value
```

Correct hardware behavior should be:

```text
q1 = 1
q2 = 0
```

because:
- q2 captures previous q1 value

---

- Result

```text
Simulation behavior
        ≠
Actual hardware behavior
```

This becomes:
- synthesis vs simulation mismatch

---

- ✅ Correct Sequential Coding Style

Use:

```verilog
<=
```

for sequential logic.

---

- ✅ Correct Version Using Non-Blocking

```verilog
module nonblocking_example(
    input clk,
    input d,
    output reg q1,
    output reg q2
);

always @(posedge clk) begin
    q1 <= d;
    q2 <= q1;
end

endmodule
```

---

- 🧠 What Happens Now?

Before clock edge:

```text
d  = 1
q1 = 0
q2 = 0
```

At clock edge:
- both RHS values evaluated first
- updates happen together later

So:

```text
q1 gets d  → 1
q2 gets old q1 → 0
```

---

- ✅ Correct Result

After one clock edge:

```text
q1 = 1
q2 = 0
```

Now:
- simulation matches hardware
- behavior becomes correct

---

- 📊 Blocking vs Non-Blocking

| Feature | Blocking (`=`) | Non-Blocking (`<=`) |
|---|---|---|
| Execution | Sequential | Parallel |
| Update Timing | Immediate | End of timestep |
| Hardware Matching | Poor for sequential logic | Correct for flip-flops |
| Mostly Used For | Combinational logic | Sequential logic |
| Can Cause Mismatch? | Yes | Less likely |

---

- 📌 Recommended Usage

#### Use Blocking (`=`) For

```verilog
always @(*)
```

Combinational logic.

Example:
- mux
- decoder
- encoder
- ALU combinational blocks

---

#### Use Non-Blocking (`<=`) For

```verilog
always @(posedge clk)
```

Sequential logic.

Example:
- flip-flops
- counters
- FSM registers
- pipelines

---

#### Simple Rule Used in Industry

```text
Combinational Logic
        ↓
Use Blocking (=)

Sequential Logic
        ↓
Use Non-Blocking (<=)
```

---

- Why Blocking Causes Problems in Sequential Logic?

Because blocking assignments:
- update values immediately
- later statements use updated values
- execution becomes software-like

But real hardware:
- updates all flip-flops simultaneously

So simulation may behave differently from actual synthesized hardware.

---

### 🧠 Final Understanding

- Blocking Assignment (`=`)

```text
Line-by-line immediate execution
```

Good for:
- combinational logic

Bad for:
- clocked sequential logic

---

- Non-Blocking Assignment (`<=`)

```text
All register updates happen together
```

Matches:
- real flip-flop hardware behavior

Best for:
- sequential logic

---

- ✅ Final Industry Practice

```verilog
always @(*)      → use =

always @(posedge clk) → use <=
```

Following this rule helps avoid:
- synthesis vs simulation mismatches
- race conditions
- incorrect RTL behavior

[⬆ Back to Repository Contents](#repository-contents)


# 8. Optimization techniques
- Optimization in Synthesis? = Optimization is the process of improving the synthesized hardware design to achieve better timing, area, power and performance while still preserving the original functionality of the RTL design.

---

### Why Optimization is Needed?

Direct RTL-to-gate conversion may produce:
- unnecessary logic
- redundant gates
- slow paths
- extra power consumption

Optimization helps remove these inefficiencies/redudant logic.

---

### Main Goals of Optimization

| Goal | Purpose |
|---|---|
| Area Optimization | Reduce hardware size |
| Timing Optimization | Improve circuit speed |
| Power Optimization | Reduce power consumption |
| Logic Optimization | Simplify hardware logic |

---

## Types of Optimization Techniques

Optimization techniques are mainly divided into:

1. Combinational Optimization
2. Sequential Optimization

### Common Combinational Optimization Techniques

#### 1.1 Constant Propagation

If a signal value is constant, the synthesis tool simplifies the logic.

Example:

```verilog
assign y = a & 1'b1;
```

Simplified into:

```verilog
assign y = a;
```

---

#### 1.2 Boolean Simplification

The synthesis tool simplifies Boolean expressions mathematically.

Example:

```verilog
assign y = (a & b) | (a & ~b);
```

Simplified into:

```verilog
assign y = a;
```

---

#### 1.3 Dead Logic Elimination

Unused logic is removed.

Example:

```verilog
wire temp;
assign temp = a & b;
```

If `temp` is never used:
- tool removes the wire
- associated gates are also removed

---

#### 1.4 Common Subexpression Optimization

Repeated logic is shared to reduce duplicate hardware.

---

#### 1.5 Mux Optimization

Synthesis tool identifies mux structures and implement optimized multiplexers.

Example:

```verilog
if(sel)
    y = i1;
else
    y = i0;
```

Recognized as:

```text
2:1 Multiplexer
```

---

### Simple Understanding of Combinational Optimization

```text
Complex Logic
      ↓
Logic Simplification
      ↓
Fewer Gates
      ↓
Smaller/Faster Hardware
```

---

### 2. Sequential Optimization

Sequential optimization improves:
- flip-flops
- registers
- counters
- FSM logic
- sequential timing paths

---

### Common Sequential Optimization Techniques

### 2.1 Register Optimization

Unnecessary or redundant registers may be removed or optimized.

---

### 2.2 Counter Optimization

Synthesis tool recognizes counters and generates efficient counter hardware.

Example:

```verilog
count <= count + 1;
```

Tool identifies:
- incrementing pattern
- optimized counter structure

---

### 2.3 Sequential Constant Propagation

If a register always stores a constant value, logic can be simplified.

Example:

```verilog
q <= 1'b0;
```

Tool may simplify dependent logic.

---

### 2.4 Flip-Flop Inference

Synthesis tool recognizes sequential RTL patterns and infers flip-flops.

Example:

```verilog
always @(posedge clk)
    q <= d;
```

Tool infers:

```text
D Flip-Flop
```

---

### 2.5 FSM Optimization

Finite State Machines (FSMs) optimization is a broader term that refers to improving the overall FSM implementation.

### FSM Optimization

```text
FSM Optimization
│
├── State Optimization
│      ├── Equivalent State Removal
│      ├── State Minimization
│
├── State Encoding Optimization
│      ├── Binary Encoding
│      ├── One-Hot Encoding
│
├── State Cloning(optimization technique where the synthesis tool creates an extra copy of hardware logic or a state to reduce delay and improve timing performance.)
│
├── Transition Optimization
│
└── Unreachable State Removal
```

### Simple Understanding of Sequential Optimization

```text
Sequential RTL
      ↓
Register/Counter/FSM Analysis
      ↓
Optimized Sequential Hardware
```

### Optimization Example Using good_mux

## RTL Code

```verilog
module good_mux(
    input  i0,
    input  i1,
    input  sel,
    output reg y
);

always @(*) begin
    if(sel)
        y = i1;
    else
        y = i0;
end

endmodule
```

---

### What Synthesis Tool Does?

Tool recognizes:

```text
This is a 2:1 Multiplexer
```

Then:
- simplifies logic
- removes unnecessary gates
- maps into optimized mux hardware

---

### Real Optimization Tradeoffs

Sometimes optimization for one parameter affects another.

Example:

| Optimization | Effect |
|---|---|
| Faster cells | Better timing but more power |
| Smaller cells | Lower area but slower timing |
| Extra buffers | Fix timing but increase area |


[⬆ Back to Repository Contents](#repository-contents)


# 9. Basics on RTL coding styles

RTL coding style is very important in digital design because coding style directly affects the synthesized hardware, timing, area, power and reliability.

Poor RTL coding style can create:
- unintended hardware
- latches
- timing issues
- synthesis vs simulation mismatches

---

### 1. Inferred Latches

- What is a Latch? = A latch is a level-sensitive storage element.

Unlike flip-flops:
- latches are transparent when enable is active and output can change immediately with input



- How Latches Get Inferred Accidentally? = Latches are usually inferred when combinational logic does NOT assign output in all possible conditions

This means:
- output must "remember" previous value
- synthesis tool adds storage element
- latch gets inferred


#### Bad Example : Inferred Latch

```verilog
module bad_latch(
    input a,
    input en,
    output reg y
);

always @(*) begin
    if(en)
        y = a;
end

endmodule
```

### If and Case Statements and Their Impact on Hardware

In Verilog, `if` and `case` statements are used to make decisions in hardware design.

But an important point is: RTL code directly creates hardware, So different coding styles create different hardware structures.

## Simple Example

```verilog
always @(*) begin

    if(sel)
        y = a;
    else
        y = b;

end
```

This creates:
```text
2:1 Multiplexer (MUX)
```

Because:
- `sel` selects between `a` and `b`

### Difference Between If and Case

| If Statement | Case Statement |
|---|---|
| Creates priority logic | Creates selection logic |
| Conditions checked one-by-one | Direct selection |
| Can become slower | Usually simpler/faster |
| Good for priority conditions | Good for mux/FSM |


### Hazards in If and Case Statements
- Missing ELSE or incomplete if
     -> this cause output must remember only the if value so the tool creates latch here, and this unwanted latch create syn-sim mismatch or timing issues.

- Missing DEFAULT in CASE --> this also cause output must remember its previous when input sel not matches to the mentioned values.

- priority hazard in if statements --> if-else creates priority logic, means else executes only if part becomes false. so this can increases delay.
   -> this because if-else performs sequential priority checking which makes hardware slowere and case statement performs parallel selection.

- overlapping case hazard --> this happens when 2 condition of case statements were matched. here we can see mismatch and unexpected output.


---

### `for` Loop → Repeating Logic

A normal `for` loop is used inside always block

It is mainly used for repeated assignments, bit operations and array operations for reducing repeptitive code.

---
### `generate for` Loop → Repeating Hardware Blocks

A `generate for` loop is used outside `always` blocks and inside of `generate` and `endgenerate` blocks.

It is mainly used for:
- creating repeated hardware structures
- instantiating multiple modules
- scalable RTL design

### Difference Between `for` Loop and `generate for` Loop

Both are used to create repeated hardware in Verilog.

But the main difference is:

```text
for loop
    ↓
Repeats logic statements

generate for loop
    ↓
Repeats hardware/module creation
```

---
[⬆ Back to Repository Contents](#repository-contents)

# 10. Final Understanding

- RTL design need's to be simulated for verifying its functional logic before it converted into netlist through synthesis
- after synthesis if Synthesis vs simulation mismatch occurs when RTL simulation behavior and synthesized hardware behavior do not match.
-  This mismatch may occur because simulators execute RTL code behaviorally, while synthesis tools convert RTL into actual hardware gates and circuits. so after synthesis need to do GLS validation (with the same testbench which was used for validating the RTL design) for cross checking the function logic.
- Optimization means improving synthesized hardware by simplifying logic, removing unnecessary hardware, improving timing and reducing area, power usage.
- while RTL coding, we have beaware of using if and case statements based on necessity

| Hazard | Cause | Hardware Impact |
|---|---|---|
| Incomplete IF | Missing `else` | Latch inference |
| Incomplete CASE | Missing cases / `default` | Latch inference |
| IF-ELSE Priority | Sequential condition checking | Slower priority logic |
| Overlapping CASE | Multiple matching conditions | Ambiguous hardware |
| X/Z Conditions | Unknown (`X`) or High-Impedance (`Z`) states | Simulation mismatch |

Therefore:
- proper RTL coding style
- good verification
- Gate-Level Simulation (GLS) are very important in digital VLSI design.


[⬆ Back to Repository Contents](#repository-contents)