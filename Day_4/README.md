# RTL Design and Synthesis Workshop – Day 4

---

# 1. Gate-Level Simulation (GLS) Essentials

## Understanding GLS

Gate-Level Simulation (GLS) is a verification methodology where the post-synthesis gate-level netlist is simulated to confirm structural and functional implementation accuracy.

It validates that the physical hardware mapping matches the logical assumptions made during RTL coding.

### Verification Objectives

- **Functional Correctness** → Ensures synthesis does not alter RTL intent.
- **Timing Behavior** → Detects setup and hold violations using SDF timing files.
- **Power Estimates** → Enables switching activity analysis for accurate power reporting.
- **Test Structures** → Verifies scan chains and DFT structures.

---

## GLS Classifications

### Functional GLS

- Zero-delay or unit-delay simulation
- Used mainly for functional validation

### Timing GLS

- Delay-annotated simulation
- Uses back-annotated timing data

---

# 2. Synthesis-Simulation Mismatch

A synthesis-simulation mismatch occurs when RTL simulation behavior differs from synthesized hardware behavior.

## Common Causes

### Non-Synthesizable Constructs

Examples:
- `initial` blocks
- `#delay` statements

### Incomplete Sensitivity Lists

Missing signals inside `always` blocks can cause incorrect simulation behavior.

### Ambiguous Case Assignments

Incomplete assignments may infer unintended latches.

---

# 3. Procedural Assignment Typing

## Blocking Statements (`=`)

### Characteristics

- Sequential execution
- Immediate assignment update
- Mostly used in combinational logic

### Example

```verilog
always @(*) begin
    a = b & c;
    d = a | e;
end
```

---

## Non-Blocking Statements (`<=`)

### Characteristics

- Parallel scheduling
- Updated at end of current timestep
- Mostly used in sequential logic

### Example

```verilog
always @(posedge clk) begin
    q <= d;
end
```

---

## Assignment Comparison Table

| Feature | Blocking (`=`) | Non-Blocking (`<=`) |
|---|---|---|
| Execution | Sequential | Parallel |
| Update Timing | Immediate | End of timestep |
| Usage | Combinational Logic | Sequential Logic |
| Hardware Inference | Logic Gates | Registers / Flip-Flops |

---

# 4. Labs: Simulation and Synthesis Verification

## Lab 1: Ternary Operator MUX

### Verilog Design (`ternary_operator_mux.v`)

```verilog
module ternary_operator_mux (
    input i0,
    input i1,
    input sel,
    output y
);

assign y = sel ? i1 : i0;

endmodule
```

### Functional Evaluation

- If `sel = 1`, output becomes `i1`
- If `sel = 0`, output becomes `i0`

---

## Lab 2: Standard MUX Synthesis

### Objective

Synthesize the ternary multiplexer using Yosys.

### Run Synthesis Script

```bash
yosys -p "
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib;
read_verilog ternary_operator_mux.v;
synth -top ternary_operator_mux;
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib;
show"
```

---

## Lab 3: Gate-Level Simulation (GLS) Execution

### Compile Netlist Simulation

```bash
iverilog -o gls_mux_sim.out /path/to/primitives.v /path/to/sky130_fd_sc_hd.v ternary_operator_mux.v testbench.v
```

### Execute Simulation

```bash
./gls_mux_sim.out
```

### Open Waveforms

```bash
gtkwave testbench.vcd
```

---

## Lab 4: Incomplete Sensitivity List Pitfalls

### Incorrect Verilog Design (`bad_mux.v`)

```verilog
module bad_mux (
    input i0,
    input i1,
    input sel,
    output reg y
);

always @(sel)
begin
    if (sel)
        y <= i1;
    else
        y <= i0;
end

endmodule
```

### Bug Analysis

#### Sensitivity Truncation

The block triggers only when `sel` changes.
Changes in `i0` and `i1` are ignored during simulation.

#### Improper Assignment Style

Non-blocking assignments (`<=`) are incorrectly used inside combinational logic.

### Corrected Code

```verilog
always @(*)
begin
    if (sel)
        y = i1;
    else
        y = i0;
end
```

---

## Lab 5: Simulating Mismatch Behaviors

### Run GLS Simulation

```bash
iverilog -o bad_mux_sim.out /path/to/primitives.v /path/to/sky130_fd_sc_hd.v bad_mux.v tb_bad_mux.v

./bad_mux_sim.out
```

---

## Lab 6: Blocking Race Conditions

### Incorrect Verilog Design (`blocking_caveat.v`)

```verilog
module blocking_caveat (
    input a,
    input b,
    input c,
    output reg d
);

reg x;

always @(*)
begin
    d = x & c;
    x = a | b;
end

endmodule
```

### Bug Analysis

Because blocking assignments execute immediately, `d` uses the old value of `x` before it gets updated.

### Corrected Code

```verilog
always @(*)
begin
    x = a | b;
    d = x & c;
end
```

---

## Lab 7: Synthesis Validation

### Run Synthesis Verification

```bash
yosys -p "
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib;
read_verilog blocking_caveat.v;
synth -top blocking_caveat;
show"
```

---

# 5. Learning Outcome

By completing this workshop, you learned:

- Gate-Level Simulation fundamentals
- Timing and synthesis verification
- Causes of synthesis-simulation mismatch
- Proper use of blocking and non-blocking assignments
- Race condition debugging techniques
- Yosys-based synthesis validation flow

---
