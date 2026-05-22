# RTL Design and Synthesis Workshop – Day 2

## Introduction to Timing Libraries

### SKY130 PDK Overview

The SKY130 PDK is an open-source Process Design Kit based on SkyWater Technology's 130nm CMOS technology. It provides foundational models and standard cell libraries essential for integrated circuit (IC) design, documenting critical timing, power, and process variation characteristics.

### Decoding `tt_025C_1v80` in the SKY130 PDK

Standard cell library names indicate specific process, voltage, and temperature (PVT) modeling conditions:

- **tt** → Typical-typical process corner
- **025C** → Operating temperature of 25°C, used to gauge temperature-dependent behavior
- **1v80** → Core supply voltage fixed at 1.8V

### Opening and Exploring the `.lib` File

The timing library configuration can be inspected using a standard text editor.

```bash
sudo apt install gedit
gedit sky130_fd_sc_hd__tt_025C_1v80.lib
```

---

## Hierarchical vs. Flattened Synthesis

### Hierarchical Synthesis

Hierarchical synthesis preserves the structural module boundaries defined within the original RTL code, processing each sub-module independently during execution. Tools like Yosys employ distinct structural passes to isolate and construct the module framework.

**Advantages**
- Drastically reduces synthesis compilation time for larger, high-density designs.
- Simplifies post-synthesis debugging and static timing analysis by preserving design boundaries.
- Enhances modular design workflows, easing integration across multi-tool environments.

**Disadvantages**
- Restricts the optimization engine from optimizing logic across module boundaries.
- Requires additional design constraints to generate thorough boundary reports.

### Flattened Synthesis

Flattened synthesis dissolves the design hierarchy completely, merging all sub-modules into a singular, unified gate-level netlist. The structural hierarchy is dissolved to expose the complete design logic to global optimizations.

**Advantages**
- Allows aggressive, cross-boundary logic optimization and gate restructuring.
- Delivers a single flat netlist file, simplifying specific backend synthesis and layout tasks.

**Disadvantages**
- Increases tool execution runtimes significantly on dense designs.
- Obscures module boundaries, making debugging, signal tracing, and error isolation highly complex.
- Demands significantly higher computational memory allocation during synthesis.

### Architectural Comparison

| Architectural Feature | Hierarchical Synthesis | Flattened Synthesis |
|---|---|---|
| Design Hierarchy | Preserved and intact | Completely dissolved and flat |
| Optimization Boundary | Restricted to individual modules | Applied globally across the whole design |
| Tool Runtime | Highly efficient for large-scale systems | Exponentially longer for large-scale systems |
| Debugging Complexity | Low (direct correlation to RTL) | High (complex signal tracing) |
| Netlist Output | Structured, modular layout | Single, interconnected logic block |
| Primary Objective | Isolation, clean reporting, and modularity | Peak area, timing, and logic optimization |

---

## Flip-Flop Coding Styles

Flip-flops are fundamental memory structures utilized to hold state data within digital hardware systems.

### Asynchronous Reset D Flip-Flop

```verilog
module dff_asyncres (
    input clk,
    input async_reset,
    input d,
    output reg q
);

always @(posedge clk or posedge async_reset) begin
    if (async_reset)
        q <= 1'b0;
    else
        q <= d;
end

endmodule
```

- **Asynchronous reset** → Instantly forces the output `q` to logic `0` on the rising edge of the reset signal, independent of the clock status.
- **Edge-triggered operation** → Samples and registers input `d` on the rising clock edge when the reset signal is inactive.

### Asynchronous Set D Flip-Flop

```verilog
module dff_async_set (
    input clk,
    input async_set,
    input d,
    output reg q
);

always @(posedge clk or posedge async_set) begin
    if (async_set)
        q <= 1'b1;
    else
        q <= d;
end

endmodule
```

- **Asynchronous set** → Overrides the clock line to drive output `q` to logic `1` immediately upon activation.

### Synchronous Reset D Flip-Flop

```verilog
module dff_syncres (
    input clk,
    input sync_reset,
    input d,
    output reg q
);

always @(posedge clk) begin
    if (sync_reset)
        q <= 1'b0;
    else
        q <= d;
end

endmodule
```

- **Synchronous reset** → The state clear condition is evaluated and applied exclusively on the active edge of the clock signal.

---

## Simulation and Synthesis Workflow

### Icarus Verilog Simulation

```bash
iverilog dff_asyncres.v tb_dff_asyncres.v
./a.out
gtkwave tb_dff_asyncres.vcd
```

### Synthesis with Yosys

```bash
yosys
read_liberty -lib /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog /path/to/dff_asyncres.v
synth -top dff_asyncres
dfflibmap -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

---

## Learning Outcome

- Decoded liberty timing file configurations (`tt`, `025C`, `1v80`).
- Investigated structural differences between hierarchical and flattened synthesis approaches.
- Implemented sequential logic behaviors using synchronous/asynchronous reset and set conditions.
- Synthesized sequential Verilog blocks using `dfflibmap` technology targeting Yosys.
