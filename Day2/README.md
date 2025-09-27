# Day 2:  RTL Design & Synthesis with Yosys

## Overview
On Day 2, I explored deeper synthesis concepts using **Yosys** and **Sky130 standard cell libraries**.  
I worked with **flip-flops (DFFs), resets (sync/async), sets, and hierarchical designs**.  
I also studied **flattening of modules, optimization techniques, and inside-library mapping**.

---

## 📂 Table of Contents
1. Inside a Liberty Library  
2. Working with Flip-Flops  
   - Synchronous Reset DFF  
   - Asynchronous Reset DFF  
   - Asynchronous Set DFF  
3. Hierarchical & Flattened Modules  
4. Optimization in Verilog  
5. My Learnings  
6. Summary  

---

## Inside a Liberty Library
- The **`.lib` file** defines timing, power, and functionality of each standard cell.  
- Contains:
  - Cell type (NAND, NOR, DFF, MUX, etc.)
  - Area & drive strength
  - Timing arcs (setup, hold, clk→q)
  - Power info  

This is the backbone of mapping RTL into technology-specific cells.

---

## Working with Flip-Flops

### 🔹 Synchronous Reset DFF
- Reset happens **synchronously with clock edge**.  
- Until the next rising edge of clock, output does not reset immediately.  
- Used in datapath pipelines where reset can wait for clock control.  

---

### 🔹 Asynchronous Reset DFF
- Reset happens **immediately**, regardless of clock.  
- Common in real hardware for safe initialization.  
- Important for circuits that need to start in a known state after power-up.  

---

### 🔹 Asynchronous Set DFF
- Output `Q` is forced to `1` immediately, independent of clock.  
- Useful in control logic and FSM initialization sequences.  

---

## Hierarchical & Flattened Modules

### Hierarchical Module Design
- A design can be broken into **submodules** (reusable blocks).  
- Example: `multiple_modules.v` contains two submodules connected internally.  
- Hierarchical representation is easier to debug and maintain.  

### Flattened Module Design
- Using `flatten` in Yosys merges all submodules into a **single module**.  
- This is often used before Place & Route (PnR) for easier optimization.  
- Flattening helps the synthesis tool optimize across module boundaries.  

---

## Optimization in Verilog

Optimization techniques in Yosys:
1. **Constant Propagation** – replaces logic with constants when inputs are fixed.  
2. **State Optimization** – removes unreachable states in FSM.  
3. **Cloning / Retiming** – duplicates or moves registers to balance timing paths.  

These ensure smaller area, higher frequency, and reduced power consumption.

---

## My Learnings
✅ Understood the role of **Liberty (.lib) files** in mapping RTL → standard cells.  
✅ Explored **different reset/set styles in DFFs** (sync vs async).  
✅ Learned to visualize and interpret **simulation waveforms (GTKWave)**.  
✅ Practiced **hierarchical design vs flattened design** in Yosys.  
✅ Gained insights into **Verilog optimization techniques**.  

---

## Summary
- Day 2 focused on **deeper synthesis concepts**: flip-flop behavior, resets/sets, and optimizations.  
- Learned **hierarchical design practices** and when to flatten modules.  
- Understood how **standard cell mapping** works with the Sky130 library.  
- Built confidence in using **Yosys + GTKWave workflow** for larger designs.  

🚀 This knowledge strengthens my foundation for **ASIC/FPGA design**.

