# RTL Design and Synthesis Workshop – Day 5

---

# 1. Conditional Logic Optimization

## Behavioral If-Else Expressions

The `if-else` statement controls conditional execution paths inside procedural blocks such as `always` blocks.

It evaluates conditions and determines which hardware logic path should execute.

### Structural Parameters

#### Condition Evaluation

Processes mathematical or single-bit expressions into:
- TRUE (`1`)
- FALSE (`0`)

#### Grouping Limits

Uses `begin ... end` blocks to group multiple statements together.

---

## Nested Conditional Structuring

Nested conditions create prioritized hardware structures.

### Example

```verilog
if (condition1) begin
    // Executes when condition1 is TRUE
end
else if (condition2) begin
    // Executes when condition2 is TRUE
end
else begin
    // Default fallback path
end
```

---

# 2. Latch Inference Pitfalls

## Unintended Latch Mechanics

A latch is inferred when an output signal is not assigned in all possible execution branches.

To preserve the previous value, synthesis tools automatically insert storage elements (latches).

---

## Flawed Design Example

### Verilog Design

```verilog
module ex (
    input wire a,
    input wire b,
    input wire sel,
    output reg y
);

always @(a, b, sel) begin
    if (sel == 1'b1)
        y = a;
end

endmodule
```

### Problem

- Missing `else` condition
- `y` retains previous state when `sel = 0`
- Causes inferred latch generation

---

## Corrected Design Example

```verilog
module ex (
    input wire a,
    input wire b,
    input wire sel,
    output reg y
);

always @(a, b, sel) begin
    case(sel)
        1'b1    : y = a;
        default : y = 1'b0;
    endcase
end

endmodule
```

### Fix Applied

- Added explicit default assignment
- Prevented latch inference

---

# 3. Labs: Conditional Branches and Case Mapping

## Lab 1: Incomplete If Constraint

### Verilog Design (`incomp_if.v`)

```verilog
module incomp_if (
    input i0,
    input i1,
    input i2,
    output reg y
);

always @(*) begin
    if (i0)
        y <= i1;
end

endmodule
```

### Hardware Evaluation

Since there is no assignment when `i0 = 0`, synthesis introduces a latch to preserve the previous value of `y`.

---

## Lab 2: Flawed Nested Conditions

### Verilog Design (`incomp_if2.v`)

```verilog
module incomp_if2 (
    input i0,
    input i1,
    input i2,
    input i3,
    output reg y
);

always @(*) begin
    if (i0)
        y <= i1;
    else if (i2)
        y <= i3;
end

endmodule
```

### Hardware Evaluation

If both `i0` and `i2` are LOW, `y` remains unassigned.

This causes another inferred latch condition.

---

## Lab 3: Fully Specified Case Structure

### Verilog Design (`comp_case.v`)

```verilog
module comp_case (
    input i0,
    input i1,
    input i2,
    input [1:0] sel,
    output reg y
);

always @(*) begin
    case(sel)
        2'b00   : y = i0;
        2'b01   : y = i1;
        default : y = i2;
    endcase
end

endmodule
```

### Hardware Evaluation

The default condition guarantees complete assignment coverage.

This maps cleanly into combinational multiplexer hardware.

---

## Lab 4: Wildcard Case Evaluation

### Verilog Design (`bad_case.v`)

```verilog
module bad_case (
    input i0,
    input i1,
    input i2,
    input i3,
    input [1:0] sel,
    output reg y
);

always @(*) begin
    case(sel)
        2'b00 : y = i0;
        2'b01 : y = i1;
        2'b10 : y = i2;
        2'b1? : y = i3;
    endcase
end

endmodule
```

### Hardware Evaluation

Using wildcard values (`?`) inside a normal `case` statement may leave logic gaps.

Undefined states can still create latch conditions.

---

## Lab 5: Asymmetric Variable Assignment

### Verilog Design (`partial_case_assign.v`)

```verilog
module partial_case_assign (
    input i0,
    input i1,
    input i2,
    input [1:0] sel,
    output reg y,
    output reg x
);

always @(*) begin
    case(sel)

        2'b00: begin
            y = i0;
            x = i2;
        end

        2'b01: begin
            y = i1;
        end

        default: begin
            x = i1;
            y = i2;
        end

    endcase
end

endmodule
```

### Hardware Evaluation

Signal `x` is not assigned during the `2'b01` condition.

This causes latch inference only for signal `x`.

---

# 4. Loops and Structural Generates

## For Loop Application

A procedural `for` loop automates repetitive assignments inside combinational or sequential blocks.

### Synthesizable Requirements

Loop boundaries must be constant values known at compile time.

---

## Example: 4-to-1 MUX Using For Loop

```verilog
module mux_4to1_for_loop (
    input wire [3:0] data,
    input wire [1:0] sel,
    output reg y
);

integer i;

always @(data, sel) begin

    y = 1'b0;

    for (i = 0; i < 4; i = i + 1) begin
        if (i == sel)
            y = data[i];
    end

end

endmodule
```

---

## Compile-Time Generate Blocks

Generate blocks replicate hardware during compilation.

They simplify scalable structural hardware creation.

### Example

```verilog
genvar i;

generate
    for (i = 0; i < 4; i = i + 1) begin : gen_loop
        and_gate and_inst (
            .a(in[i]),
            .b(in[i+1]),
            .y(out[i])
        );
    end
endgenerate
```

---

# 5. Case Study: Ripple Carry Adder (RCA)

A Ripple Carry Adder (RCA) chains multiple full adders together.

The carry output of one stage becomes the carry input of the next stage.

---

# 6. Labs: Iterative Loops and Hardware Replication

## Lab 6: 4-to-1 MUX Iteration

### Verilog Design (`mux_generate.v`)

```verilog
module mux_generate (
    input i0,
    input i1,
    input i2,
    input i3,
    input [1:0] sel,
    output reg y
);

wire [3:0] i_int;
assign i_int = {i3, i2, i1, i0};

integer k;

always @(*) begin

    y = 1'b0;

    for (k = 0; k < 4; k = k + 1) begin
        if (k == sel)
            y = i_int[k];
    end

end

endmodule
```

---

## Lab 7: Explicit Case Demultiplexer

### Verilog Design (`demux_case.v`)

```verilog
module demux_case (

    output o0, output o1, output o2, output o3,
    output o4, output o5, output o6, output o7,

    input [2:0] sel,
    input i
);

reg [7:0] y_int;

assign {o7, o6, o5, o4, o3, o2, o1, o0} = y_int;

always @(*) begin

    y_int = 8'b0;

    case(sel)

        3'b000 : y_int[0] = i;
        3'b001 : y_int[1] = i;
        3'b010 : y_int[2] = i;
        3'b011 : y_int[3] = i;
        3'b100 : y_int[4] = i;
        3'b101 : y_int[5] = i;
        3'b110 : y_int[6] = i;
        3'b111 : y_int[7] = i;

    endcase

end

endmodule
```

---

## Lab 8: Loop-Driven Demultiplexer

### Verilog Design (`demux_generate.v`)

```verilog
module demux_generate (

    output o0, output o1, output o2, output o3,
    output o4, output o5, output o6, output o7,

    input [2:0] sel,
    input i
);

reg [7:0] y_int;

assign {o7, o6, o5, o4, o3, o2, o1, o0} = y_int;

integer k;

always @(*) begin

    y_int = 8'b0;

    for (k = 0; k < 8; k = k + 1) begin
        if (k == sel)
            y_int[k] = i;
    end

end

endmodule
```

---

## Lab 9: Scalable 8-bit RCA Array

### Verilog Design (`rca.v`)

```verilog
module rca (
    input [7:0] num1,
    input [7:0] num2,
    output [8:0] sum
);

wire [7:0] int_sum;
wire [7:0] int_co;

genvar i;

generate

    for (i = 1; i < 8; i = i + 1) begin : adder_block

        fa u_fa_1 (
            .a(num1[i]),
            .b(num2[i]),
            .c(int_co[i-1]),
            .co(int_co[i]),
            .sum(int_sum[i])
        );

    end

endgenerate

fa u_fa_0 (
    .a(num1[0]),
    .b(num2[0]),
    .c(1'b0),
    .co(int_co[0]),
    .sum(int_sum[0])
);

assign sum[7:0] = int_sum;
assign sum[8]   = int_co[7];

endmodule
```

---

## Full Adder Primitive (`fa.v`)

```verilog
module fa (
    input a,
    input b,
    input c,
    output co,
    output sum
);

assign {co, sum} = a + b + c;

endmodule
```

---

# 7. Learning Outcome

By completing this workshop, you learned:

- How to avoid inferred latch generation
- Proper use of `if-else` and `case` statements
- Loop-based hardware replication techniques
- Generate block implementation methods
- Scalable RCA hardware construction
- Iterative combinational hardware design concepts

---
