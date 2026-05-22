# RTL Design and Synthesis Workshop – Day 3

---

# 1. Combinational Optimization Techniques

## Constant Propagation

Constant propagation is an optimization technique where compile-time constants are substituted directly into expressions to eliminate redundant variables.

Replacing known variables with static values allows the synthesis engine to:
- simplify gate structures,
- remove dead logic,
- reduce hardware complexity.

### Benefits

- **Reduced Complexity** → Cleaner logic structures and lower gate count.
- **Performance Improvement** → Reduced propagation delay and improved timing.
- **Resource Optimization** → Eliminates unnecessary logic resources.

---

## State Optimization

State optimization improves the efficiency of Finite State Machine (FSM) implementations.

### Key Implementation Tasks

- **State Reduction** → Removes redundant or equivalent states.
- **State Encoding** → Uses optimized encoding methods such as:
  - One-Hot
  - Binary
  - Gray Encoding
- **Logic Minimization** → Reduces combinational hardware complexity.
- **Power Optimization** → Uses techniques like clock gating to reduce dynamic power.

---

# 2. Sequential Optimization Techniques

## Cloning

Cloning duplicates highly loaded logic cells to reduce fan-out delay and improve timing performance.

### Workflow

1. Identify highly loaded critical cells.
2. Duplicate the required logic block.
3. Split the fan-out load between original and cloned cells.
4. Perform placement and rerouting.
5. Re-check timing closure.

---

## Retiming

Retiming shifts registers across combinational logic to reduce critical path delay without changing functionality.

### Workflow

1. **Graph Representation** → Model the design as a timing graph.
2. **Register Repositioning** → Move registers across logic stages.
3. **Constraint Analysis** → Verify timing and functionality.
4. **Optimization** → Improve clock frequency and timing.

---

# 3. Labs: Combinational Optimization

## Lab 1: Ternary Conditional Optimization

### Verilog Design (`opt_check.v`)

```verilog
module opt_check (
    input a,
    input b,
    output y
);

assign y = a ? b : 1'b0;

endmodule
```

### Functional Evaluation

- If `a = 1`, output follows `b`
- If `a = 0`, output becomes `0`

### Optimization Command

Execute between `synth -top` and `abc -liberty`:

```bash
opt_clean -purge
```

---

## Lab 2: Direct Constant Mapping

### Verilog Design (`opt_check2.v`)

```verilog
module opt_check2 (
    input a,
    input b,
    output y
);

assign y = a ? 1'b1 : b;

endmodule
```

### Functional Evaluation

Implements OR-style mux logic:
- When `a = 1`, output becomes logic HIGH.
- Otherwise output follows `b`.

---

## Lab 3: Multiplexer Optimization Test

### Verilog Design (`opt_check3.v`)

```verilog
module opt_check3 (
    input a,
    input b,
    output y
);

assign y = a ? 1'b1 : b;

endmodule
```

### Functional Evaluation

Implements a simple 2:1 multiplexer structure.

---

## Lab 4: Nested Logic Pruning

### Verilog Design (`opt_check4.v`)

```verilog
module opt_check4 (
    input a,
    input b,
    input c,
    output y
);

assign y = a ? (b ? (a & c) : c) : (!c);

endmodule
```

### Functional Evaluation

The synthesis engine removes redundant logic and simplifies the expression to:

```verilog
y = a ? c : !c;
```

---

# 4. Labs: Sequential Optimization

## Lab 5: Fixed Value Asynchronous Register

### Verilog Design (`dff_const1.v`)

```verilog
module dff_const1 (
    input clk,
    input reset,
    output reg q
);

always @(posedge clk, posedge reset)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= 1'b1;
end

endmodule
```

### Functional Evaluation

- Reset forces output to `0`
- Otherwise the flip-flop continuously stores `1`

---

## Lab 6: Constant Logic Simplification

### Verilog Design (`dff_const2.v`)

```verilog
module dff_const2 (
    input clk,
    input reset,
    output reg q
);

always @(posedge clk, posedge reset)
begin
    if (reset)
        q <= 1'b1;
    else
        q <= 1'b1;
end

endmodule
```

### Functional Evaluation

Since both conditions assign logic HIGH, synthesis removes the register entirely and ties the output directly to VCC.

---

# 5. Learning Outcome

By completing this workshop, you learned:

- Constant propagation techniques
- FSM optimization strategies
- Cell cloning and fan-out balancing
- Sequential retiming concepts
- Yosys-based synthesis optimization flows

---
