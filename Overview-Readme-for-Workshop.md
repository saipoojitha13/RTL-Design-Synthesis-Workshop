#  RTL-Design-to-Synthesis-using-SKY130 Workshop

Top-level repository for learning and practicing the complete RTL-to-Synthesis flow using open-source EDA tools and the SKY130 standard-cell library.

👨‍💻 Author: **Saipoojitha**

---

# 📖 Overview

This repository contains hands-on learning material, notes, simulations, synthesis examples, waveform analysis, and practical lab work related to:

- RTL Design
- Verilog HDL
- RTL Simulation
- Gate-Level Simulation (GLS)
- Synthesis using Yosys
- SKY130 Standard Cells
- Liberty Files (`.lib`)
- Timing Concepts
- Slack Analysis
- RTL Coding Styles
- Synthesis vs Simulation Mismatch

The goal of this repository is to build strong fundamentals in digital VLSI design using open-source tools.

---

# 🛠️ Prerequisites

Before starting this workshop, it is recommended to have:

- Basic understanding of Digital Electronics
- Basic knowledge of Verilog HDL
- Familiarity with Linux terminal commands

---

# ⚙️ Lab Setup Process

Install the following open-source EDA tools:

| Tool | Purpose |
|---|---|
| `iverilog` | RTL & GLS Simulation |
| `gtkwave` | Waveform Viewer / Debugging |
| `yosys` | RTL Synthesis |

---

# 💻 Tool Installation Commands

### Install Icarus Verilog

```bash
sudo apt update
sudo apt install iverilog
```
---
### Install GTKWave

```bash
sudo apt install gtkwave
```
---

### Install Yosys

```bash
sudo apt install yosys
```

---

# 📥 Clone Workshop Reference Repository

```bash
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```

This repository provides the required workshop files and SKY130-related practice data.

---

# 📚 Major Concepts Covered

1. Spec & RTL Understanding  
2. Need for Testbench and RTL Simulation  
3. Synthesizer and Synthesis Flow  
4. RTL vs Gate-Level Netlist Functional Equivalence  
5. Liberty File, Cell Variants, Slack & Cell Selection  
6. Synthesis vs Simulation Mismatch  
7. Blocking vs Non-Blocking Assignments  
8. RTL Coding Styles & Combinational/Sequential Optimization  

---

# 📂 Repository Structure

```text
RTL-Design-to-Synthesis-using-SKY130/
│
├── README.md
├── GLS-Simulation/
├── Simulation-Lab-Work-Snaps/
├── Synthesis-Lab-Work-Snaps/
└── Additional-Notes/
```

---

## 📁 Repository Navigation

| Folder/File | Description |
|---|---|
| [README.md](./README.md) | Overview and documentation of the repository |
| [GLS-Simulation](./GLS-Simulation) | Gate-Level Simulation (GLS) examples and analysis |
| [Simulation-Lab-Work-Snaps](./Simulation-Lab-Work-Snaps) | RTL simulation screenshots and waveform analysis |
| [Synthesis-Lab-Work-Snaps](./Synthesis-Lab-Work-Snaps) | Yosys synthesis screenshots and reports |
| [Additional-Notes](./Additional-Notes) | Supporting notes and learning material |

---

# 🔄 Basic Design Flow Covered

```text
Specification
      ↓
RTL Design
      ↓
Testbench Development
      ↓
RTL Simulation
      ↓
Synthesis using Yosys
      ↓
Gate-Level Netlist
      ↓
Gate-Level Simulation (GLS)
      ↓
Waveform Analysis
```

---

# 🎯 Objectives of this Repository

- Learn RTL design fundamentals
- Understand simulation and synthesis flow
- Explore SKY130 standard-cell libraries
- Understand timing concepts and slack
- Practice debugging using GTKWave
- Understand synthesis optimization concepts
- Build strong VLSI front-end fundamentals

---

# 🤝 Contributions

This repository is mainly created for learning and educational purposes. Suggestions and improvements are always welcome.

---

# ⭐ Final Note

This repository is continuously updated with:
- new RTL examples
- synthesis experiments
- waveform analysis
- timing concepts
- debugging techniques
- practical VLSI learning notes

---