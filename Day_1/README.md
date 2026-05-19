# RTL Design and Synthesis Workshop – Day 1

## Topics Covered
- Introduction to RTL Design
- Simulator
- Design and Testbench
- Icarus Verilog Flow
- GTKWave
- Yosys Introduction
- Gate Libraries
- Technology Mapping
- Netlist Generation

---

# 1. Simulator

RTL design is verified using simulation.

A simulator checks whether the RTL code follows the required specification.

For this workshop:

- **iverilog** → used for simulation
- **gtkwave** → used for waveform viewing

---

# 2. Design

Design is the actual Verilog RTL code implementing the intended functionality.

Example:
- Multiplexer
- Decoder
- Flip-flop
- Counter

---

# 3. Testbench

A testbench applies stimulus (test vectors) to the design and observes outputs.

Functions of testbench:
- Generates inputs
- Checks outputs
- Helps verify correctness

---

# 4. How Simulator Works

- Simulator monitors input changes
- Whenever input changes, output is evaluated
- No input change → no output change

---

# 5. Testbench Architecture

Testbench contains:

- Stimulus Generator
- DUT (Design Under Test)
- Output Observer

---

# 6. Iverilog Simulation Flow

Flow:

Design + Testbench
→ iverilog
→ VCD file generated
→ GTKWave displays waveform

---

# 7. Lab: Simulating 2:1 Multiplexer

## Clone Repository

```bash
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```

```bash
cd sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

---

# 8. Install Required Tools

```bash
sudo apt install iverilog
```

```bash
sudo apt install gtkwave
```

---

# 9. Compile Design

```bash
iverilog good_mux.v tb_good_mux.v
```

---

# 10. Run Simulation

```bash
./a.out
```

---

# 11. Open Waveform

```bash
gtkwave tb_good_mux.vcd
```

---

# 12. Verilog Code Analysis

## good_mux.v

```verilog
module good_mux (input i0, input i1, input sel, output reg y);

always @ (*)
begin
    if(sel)
        y <= i1;
    else
        y <= i0;
end

endmodule
```

---

## Working

Inputs:
- i0
- i1
- sel

Output:
- y

Logic:
- sel = 1 → y = i1
- sel = 0 → y = i0

---

# 13. Introduction to Yosys

Yosys is an open-source synthesis tool.

Functions:
- RTL synthesis
- Optimization
- Technology mapping
- Gate-level netlist generation

---

# 14. Gate Library (.lib)

A `.lib` file contains timing and gate information.

Example:
- AND gate
- OR gate
- MUX gate
- NOT gate

Different gate flavors exist for:
- speed
- power
- area
- drive strength

---

# 15. Yosys Synthesis Flow

## Start Yosys

```bash
yosys
```

---

## Read Liberty File

```bash
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
```

---

## Read Verilog Design

```bash
read_verilog good_mux.v
```

---

## Synthesize Design

```bash
synth -top good_mux
```

---

## Technology Mapping

```bash
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
```

---

## Show Gate-Level Netlist

```bash
show
```

---

# 16. Write Synthesized Netlist

## Write Verilog Netlist

```bash
write_verilog good_mux_netlist.v
```

---

## Open Netlist

```bash
!gvim good_mux_netlist.v
```

---

## Remove Attributes

```bash
write_verilog -noattr good_mux_netlist.v
```

---

## Open Clean Netlist

```bash
!gvim good_mux_netlist.v
```

---

# 17. Learning Outcome

- Learned RTL simulation flow
- Understood testbench concept
- Simulated Verilog using iverilog
- Viewed waveforms using GTKWave
- Learned Yosys synthesis flow
- Understood liberty files
- Generated synthesized netlist
- Learned basic Linux workflow

---
