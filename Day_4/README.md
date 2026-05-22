# RTL Design and Synthesis Workshop – Day 4

## 1. Gate-Level Simulation (GLS) Essentials

### Understanding GLS

Gate-Level Simulation (GLS) is a verification methodology where the post-synthesis gate-level netlist is simulated to confirm structural and functional implementation accuracy. It validates that the physical hardware mapping matches the logical assumptions made during RTL coding.

**Verification Objectives:**
- **Functional Correctness** → Ensures the synthesis compiler did not alter intent.
- **Timing Behavior** → Identifies real-world timing violations like setup and hold errors using Standard Delay Format (SDF) files.
- **Power Estimates** → Enables switching activity analysis for high-accuracy power reporting.
- **Test Structures** → Confirms the behavior of Design For Testability (DFT) networks and internal scan chains.

---

### GLS Classifications

- **Functional GLS** → Zero-delay or unit-delay logical simulation focused entirely on functional state tracking.
- **Timing GLS** → Full delay-annotated simulation utilizing back-annotated physical interconnect timing data.

---

## 2. Synthesis-Simulation Mismatch

A synthesis-simulation mismatch happens when pre-synthesis RTL behavioral simulations diverge from the post-synthesis gate-level simulation or physical hardware execution.

**Common Causes:**
- **Non-Synthesizable Constructs** → Including algorithmic testbench structures (`initial`, `#delay` statements) in production hardware code.
- **Incomplete Sensitivity Lists** → Leaving essential feedback signals out of behavioral evaluation blocks, triggering simulation latch behaviors.
- **Ambiguous Case Assignments** → Writing incomplete logic loops that force synthesis engines to infer unintended storage latches.

---

## 3. Procedural Assignment Typing

### Blocking Statements (`=`)

- **Syntax** → `=`
- **Execution Profile** → Sequential evaluation; blocks any subsequent statement lines until the current equation is fully executed.
- **Primary Use Case** → Combinational block descriptions (`always @(*)`) and internal variable definitions.

---

### Non-Blocking Statements (`<=`)

- **Syntax** → `<=`
- **Execution Profile** → Scheduled concurrent evaluation; captures values simultaneously and updates states at the end of the current time step.
- **Primary Use Case** → Sequential logic modeling blocks tied to specific clock boundaries (`always @(posedge clk)`).

---

### Architectural Assignment Comparison

| Core Feature | Blocking Assignments (`=`) | Non-Blocking Assignments (`<=`) |
| :--- | :--- | :--- |
| **Operator Syntax** | `=` | `<=` |
| **Execution Mechanics** | Immediate, step-by-step procedural order | Scheduled parallel updates at timestep end |
| **State Resolution** | Instant updates impact following statements | Evaluated concurrently, minimizing order reliance |
| **Design Domain** | Combinational equations / Temp math lines | Sequential storage registers / Edge flip-flops |
| **Hardware Extrapolation** | Infers combinational logic gates | Infers hardware-bound sequential registers |

---

## 4. Labs: Simulation and Synthesis Verification

### Lab 1: Ternary Operator MUX

Verilog Design (`ternary_operator_mux.v`):
```verilog
module ternary_operator_mux (input i0, input i1, input sel, output y);

assign y = sel ? i1 : i0;

endmodule
Functional Evaluation:

Implements a direct, explicit data route where output y selects data line i1 when control sel is high, defaulting to i0 when low.

Lab 2: Standard MUX Synthesis
Invoke the Yosys compilation script to synthesize the ternary multiplexer structure into target hardware cells.

Run synthesis script:

Bash
yosys -p "read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib; read_verilog ternary_operator_mux.v; synth -top ternary_operator_mux; abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib; show"
Lab 3: Gate-Level Simulation (GLS) Execution
Compile the functional standard cell libraries alongside the gate netlist and stimulus testbench.

Compile netlist simulation:

Bash
iverilog -o gls_mux_sim.out /path/to/primitives.v /path/to/sky130_fd_sc_hd.v ternary_operator_mux.v testbench.v
Execute simulation binary:

Bash
./gls_mux_sim.out
Open visual waveforms:

Bash
gtkwave testbench.vcd
Lab 4: Incomplete Sensitivity List Pitfalls
Verilog Design (bad_mux.v):

Verilog
module bad_mux (input i0, input i1, input sel, output reg y);

always @ (sel)
begin
    if (sel)
        y <= i1;
    else 
        y <= i0;
end

endmodule
Bug Analysis:

Sensitivity Truncation → The block only triggers when sel changes state. Changes on data inputs i0 or i1 are ignored by simulators, while synthesis engines infer combinational gates—generating a severe simulation-synthesis divergence.

Improper Assignment Style → Employs sequential non-blocking operators (<=) inside a purely combinational block.

Remediated Code Block:

Verilog
always @ (*)
begin
    if (sel)
        y = i1;
    else
        y = i0;
end
Lab 5: Simulating Mismatch Behaviors
Run Gate-Level Simulation targeting the flawed bad_mux netlist to observe look-ahead behavioral mismatch errors and gate warning output inside GTKWave.

Execute mismatch simulation:

Bash
iverilog -o bad_mux_sim.out /path/to/primitives.v /path/to/sky130_fd_sc_hd.v bad_mux.v tb_bad_mux.v
./bad_mux_sim.out
Lab 6: Blocking Race Conditions
Verilog Design (blocking_caveat.v):

Verilog
module blocking_caveat (input a, input b, input c, output reg d);

reg x;

always @ (*)
begin
    d = x & c;
    x = a | b;
end

endmodule
Bug Analysis:

Because blocking assignments evaluate instantly, output d resolves using the stale, previous-evaluation value of register x, rather than updating with the current logic value of a | b.

Remediated Code Block:

Verilog
always @ (*)
begin
    x = a | b;
    d = x & c;
end
Lab 7: Synthesis Validation of Race Constraints
Synthesize the updated race-free code structure using Yosys to verify that intermediate logic evaluations resolve to clean, hazard-free physical gates.

Run synthesis verification:

Bash
yosys -p "read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib; read_verilog blocking_caveat.v; synth -top blocking_caveat; show"
5. Learning Outcome
Maintained clean simulation-synthesis alignment by applying rigid hardware description practices.

Isolated gate structural delays and timing constraints through comprehensive Gate-Level Simulations.

Corrected race conditions caused by poor procedural expression order inside combinational always blocks.

Applied blocking (=) and non-blocking (<=) syntax constraints correctly across complex digital design architectures.
