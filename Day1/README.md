# Day 1 — Verilog RTL Design & Synthesis 

---

## Table of Contents

1. [What We’re Doing](#what-were-doing)
2. [Tooling & Quick Setup](#tooling--quick-setup)
3. [Concepts: Simulator, Design, Testbench](#concepts-simulator-design-testbench)
4. [Icarus Verilog: Simulation Flow](#icarus-verilog-simulation-flow)
5. [Lab: Multiplexer (RTL + TB + Waveform)](#lab-multiplexer-rtl--tb--waveform)
6. [File Structure & Testbench Nature](#file-structure--testbench-nature)
7. [Yosys & Gate Libraries](#yosys--gate-libraries)
8. [Synthesis Flow in Yosys](#synthesis-flow-in-yosys)
9. [Netlist Generation & Verification](#netlist-generation--verification)
10. [Timing Concepts: Setup • Hold • Delay](#timing-concepts-setup--hold--delay)
11. [Constraints in Synthesis](#constraints-in-synthesis)
12. [Useful Yosys Commands Recap](#useful-yosys-commands-recap)
13. [Pro Tips & Common Pitfalls](#pro-tips--common-pitfalls)
14. [My Learnings](#my-learnings)
15. [Summary](#summary)
16. [Repo Layout](#repo-layout)

---

## What We’re Doing

* **RTL simulation** with Icarus and GTKWave.
* **Synthesis** with Yosys, mapping to Sky130 library cells.
* **Netlist generation** and visualization.
* **Verification** of the netlist using the same testbench.
* Learning **timing basics** (setup/hold, propagation delay).
* Understanding why `.lib` files contain multiple *flavors* of the same cell.


---

## Tooling & Quick Setup

* **Icarus Verilog** (`iverilog`): compile & run Verilog.
* **GTKWave**: view waveforms from `.vcd`.
* **Yosys**: perform synthesis.
* **Graphviz**: render schematics.
* **Sky130 library**: Liberty `.lib` containing cells.

```bash
sudo apt update
sudo apt install iverilog gtkwave yosys graphviz
```

---

## Concepts: Simulator, Design, Testbench

* **Design (DUT):** RTL description of hardware.
* **Testbench:**

  * No primary inputs/outputs in our lab TB.
  * Provides *stimulus generation* only.
  * In this example, no observer logic — instead, GTKWave is used to view results.
* **Simulator:** Icarus compiles, runs, dumps `.vcd`.

🎯 **Image placeholder:** DUT–Testbench–Simulator interaction

---

## Icarus Verilog: Simulation Flow

```bash
iverilog good_mux.v tb_good_mux.v -o a.out
./a.out      # generates tb_good_mux.vcd
```

View waveform:

```bash
gtkwave tb_good_mux.vcd
```

📝 The `.vcd` dump is created using `$dumpfile` and `$dumpvars` inside the testbench.

🎯 **Image placeholder:** GTKWave output

---

## Lab: Multiplexer (RTL + TB + Waveform)

* Design: **2:1 mux** (`good_mux.v`).
* Inputs: `i0`, `i1`, `sel`.
* Output: `y`.
* Behavior: `sel=1 → y=i1`, else `y=i0`.

🎯 **Image placeholder:** waveform showing sel toggling between inputs

---

## File Structure & Testbench Nature

From my notes:

* **Folder setup:**

  * `mux/` → contains mux RTL.
  * `tb/` → contains testbench.
* Testbench:

  * No primary inputs or outputs.
  * Generates stimulus internally.
  * No observer process; verification is done visually through GTKWave.

📝 *Curious fact:* Even though RTL is behavioral and netlist is structural, **the testbench remains the same** since primary I/Os don’t change.

---

## Yosys & Gate Libraries

* **Netlist definition:** structural design representation using standard cells from `.lib`.
* `.lib` contents:

  * Standard cell models with timing/area/power.
  * Multiple flavors of same cell: some faster (for setup), some slower (for hold).
* **Propagation delay:** depends on input transition + output capacitance.
* Wider transistors = more drive = faster switching but higher power/area.

🎯 **Image placeholder:** library corner diagram

---

## Synthesis Flow in Yosys

```tcl
# Inside Yosys
yosys
read_liberty -lib lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog verilog/good_mux.v
synth -top good_mux
# Map DFFs if sequential
dfflibmap -liberty lib/sky130_fd_sc_hd__tt_025C_1v80.lib
# Technology mapping
abc -liberty lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr build/good_mux_netlist.v
```

📝 *Note:* The synthesizer first runs a **syntax check**, then proceeds with mapping.

🎯 **Image placeholder:** Yosys schematic output

---

## Netlist Generation & Verification

* Generate netlist with:

```tcl
write_verilog -noattr build/good_mux_netlist.v
```

* Why `-noattr`: strips Yosys-specific attributes like `(* keep *)` or `(* src = … *)`.
* Simulation uses the same TB:

```bash
iverilog build/good_mux_netlist.v verilog/tb_good_mux.v -o a.out
./a.out
gtkwave tb_good_mux.vcd
```

🎯 **Image placeholder:** Netlist schematic

---

## Timing Concepts: Setup • Hold • Delay

* **Setup time:** data must arrive before clock edge. Fixed using *fast cells*.
* **Hold time:** data must stay stable after clock edge. Fixed using *slower cells*.
* **Propagation delay:** input→output delay. Longer if load capacitance is high.

📝 Wider transistors → better current sourcing → faster charging → reduced delay.

🎯 **Image placeholder:** setup/hold diagram

---

## Constraints in Synthesis

* We must **guide** the synthesizer with constraints, otherwise it may pick arbitrary flavors.
* Examples:

  * Clock period (target frequency).
  * Input arrival/Output required times.

---

## Useful Yosys Commands Recap

* `read_verilog <file>` → load RTL.
* `read_liberty -lib <file>` → load standard-cell library.
* `synth -top <module>` → perform synthesis.
* `dfflibmap -liberty <file>` → map FFs.
* `abc -liberty <file>` → technology map comb logic.
* `write_verilog <file>` → dump netlist.
* `show <module>` → schematic visualization.

---

## Pro Tips & Common Pitfalls

* **Module naming:** `synth -top` requires *module name* (not filename).
* **Simulation vs synthesis:** both can reuse the same TB.
* **Flattening:** remove hierarchy with `flatten` before netlist export if needed.
* **Constant propagation:** e.g., `a & 0` optimized to `0`.
* **Sequential logic:** needs `dfflibmap` step.

---

## My Learnings

* Learned how **simulation and synthesis complement each other**: simulation checks functionality, synthesis prepares the design for real hardware.
* Understood that **the same testbench works for RTL and netlist verification** since top-level I/Os don’t change.
* Saw how `.lib` files provide **multiple flavors** of cells to help meet setup and hold requirements.
* Learned **propagation delay fundamentals** and how drive strength & capacitance impact timing.
* Understood that **constraints are critical** to guide the synthesis tool towards correct gate selection.
* Gained exposure to **optimization techniques** like constant propagation, cloning, retiming, and state optimization.
* Understood that synthesis is the **process**, while the netlist is the **output artifact**.

---

## Summary

In this session, I:

* Ran simulations with Icarus and viewed outputs in GTKWave.
* Built and understood a **2:1 multiplexer** in RTL and verified its behavior.
* Learned Yosys commands and how to synthesize RTL into a **technology-mapped netlist**.
* Verified synthesis correctness by reusing the same testbench.
* Understood **setup, hold, propagation delay** concepts and why PVT corners matter.
* Learned about **constraints** and their role in guiding synthesis.
* Explored advanced optimization ideas like **constant propagation, cloning, and retiming**.

🎯 **Image placeholder:** learning highlights collage

---

## Repo Layout

```
my-rtl-workshop/
├─ README.md
├─ verilog/
│  ├─ good_mux.v
│  └─ tb_good_mux.v
├─ lib/
│  └─ sky130_fd_sc_hd__tt_025C_1v80.lib
├─ build/
│  └─ good_mux_netlist.v
└─ images/
   ├─ overview_flow.png
   ├─ gtkwave_good_mux.png
   ├─ yosys_show_mux.png
   ├─ setup_hold.png
   └─ summary.png
```

