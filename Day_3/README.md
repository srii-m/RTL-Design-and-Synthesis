# RTL Design and Synthesis Workshop – Day 3

## 
1. Combinational Optimization Techniques

### Constant Propagation

Constant propagation is an optimization technique where compile-time constants are substituted directly into expressions to eliminate redundant variables. Replacing known variables with their static values allows the synthesis engine to optimize gate structures and eliminate dead logic.

**Benefits:**
- **Reduced Complexity** → Creates cleaner logic structures, shrinking the overall gate count and circuit area.
- **Performance Improvement** → Decreases routing propagation delays and minimizes setup times.
- **Resource Optimization** → Limits the allocation of unnecessary logic components and sequential elements.

---

### State Optimization

State optimization streamlines Finite State Machine (FSM) models to achieve maximum layout efficiency in hardware designs.

**Key Implementation Tasks:**
- **State Reduction** → Eliminates redundant and equivalent states through minimization algorithms.
- **State Encoding** → Maps state assignments to optimal structural styles (e.g., One-Hot, Binary, Gray encoding).
- **Logic Minimization** → Combines combinational logic equations to minimize hardware layout demands.
- **Power Optimization** → Lowers dynamic power consumption through specialized architectural techniques like clock gating.

---

## 
2. Sequential Optimization Techniques

### Cloning

Cloning replicates heavily loaded logic cells or sub-modules to alleviate severe fan-out constraints, improve power distribution, and meet tight timing requirements along critical paths.

**Workflow:**

## 
1. Pinpoint highly loaded, path-critical cells using static timing analysis tools.

## 
2. Duplicate the target gate or logic block within the netlist.

## 
3. Divide the original fan-out load between the source and cloned cells to reduce drive stress.

## 
4. Execute localized cell placement and signal rerouting.

## 
5. Re-evaluate post-route timing profiles to confirm setup and hold closure.

---

### Retiming

Retiming shifts register positions across combinational logic boundaries to minimize the critical path delay without altering the functional output of the system.

**Workflow:**

## 
1. **Graph Representation** → Models the digital design network as a directed graph where edges represent logic delays.

## 
2. **Register Repositioning** → Shifts flip-flops forward or backward across logic gates to equalize path delays.

## 
3. **Constraints Analysis** → Verifies that timing bounds and functional system integrity remain structurally unchanged.

## 
4. **Optimization** → Restructures register locations to scale down clock period requirements and optimize dynamic power.

---

## 
3. Labs: Combinational Optimization

### Lab 1: Ternary Conditional Optimization

Verilog Design (`opt_check.v`):
```verilog
module opt_check (input a , input b , output y);

assign y = a ? b : 1'b0;

endmodule
```

**Functional Evaluation:**

Evaluates input a: if high, passes signal b; if low, ties output y to ground.

Optimization Command Insertion (Execute between synth -top and abc -liberty):

```bash
```bash
opt_clean -purge
```
Lab 2: Direct Constant Mapping
Verilog Design (opt_check
2.v):

```verilog
module opt_check2 (input a , input b , output y);

assign y = a ? 1'b1 : b;

endmodule
```

**Functional Evaluation:**

Emulates OR-configured muxing logic: pulls y directly to logic 1 when a is asserted, otherwise passes input b.

Lab 3: Multiplexer Optimization Test
Verilog Design (opt_check
3.v):

```verilog
module opt_check3 (input a , input b , output y);

assign y = a ? 1'b1 : b;

endmodule
```

**Functional Evaluation:**

Implements a basic 2-to-1 multiplexing structure mapping output to a static state based on the value of condition line a.

Lab 4: Nested Logic Pruning
Verilog Design (opt_check
4.v):

```verilog
module opt_check4 (input a , input b , input c , output y);

assign y = a ? (b ? (a & c) : c) : (!c);

endmodule
```

**Functional Evaluation:**

Processes multiple inputs through layered ternary parameters. The synthesis engine automatically prunes redundant checks (b and the a & c term when a is verified high), reducing the logic down to a simple 2-input XOR-type expression:
y = a ? c : !c

## 
4. Labs: Sequential Optimization
Lab 5: Fixed Value Asynchronous Register
Verilog Design (dff_const
1.v):

```verilog
module dff_const1 (input clk, input reset, output reg q);

always @(posedge clk, posedge reset)
begin
    if(reset)
        q <= 1'b0;
    else
        q <= 1'b1;
end

endmodule
```

**Functional Evaluation:**

Instantiates a D flip-flop configured with an asynchronous reset to ground (1'b0). During standard operation, the register continuously latches a constant logic 1 on every rising clock edge.

Lab 6: Constant Logic Simplification
Verilog Design (dff_const
2.v):

```verilog
module dff_const2 (input clk, input reset, output reg q);

always @(posedge clk, posedge reset)
begin
    if(reset)
        q <= 1'b1;
    else
        q <= 1'b1;
end

endmodule
```

**Functional Evaluation:**

Synthesizes a sequential block where both the reset path and standard evaluation path route to an identical logic high state (1'b1). The synthesis compiler optimizes out the register entirely, hardwiring output q directly to the VCC rail.

## 
5. Learning Outcome
Applied constant propagation approaches to scale down redundant logic and gate counts.

Evaluated methods for structural FSM state minimization and low-power encoding strategies.

Analyzed cell cloning mechanics and loads balancing to resolve setup and hold violations.

Implemented sequential retiming steps to adjust clock boundaries across complex logic networks.

Simulated and synthesized multiple optimization configurations utilizing Yosys logic sweeping scripts.
