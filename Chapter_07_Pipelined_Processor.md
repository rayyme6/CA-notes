# Chapter 7 — Pipelined RISC-V Processor

## 1. Main Objective

This is one of the most important final-exam topics.

The main idea is:

> A pipelined processor divides instruction execution into stages so multiple instructions can execute at the same time.

Pipelining improves throughput, but hazards can create stalls and flushes.

---

# 2. Five Pipeline Stages

The classic 5-stage RISC-V pipeline is:

| Stage | Name | Main work |
|---|---|---|
| IF | Fetch | fetch instruction using PC |
| ID | Decode | decode instruction and read registers |
| EX | Execute | ALU operation or address calculation |
| MEM | Memory | data memory read/write |
| WB | Writeback | write result to register file |

Pipeline registers are placed between stages:

```text
IF/ID -> ID/EX -> EX/MEM -> MEM/WB
```

Signals are often named by stage:

```text
PCF, PCD, PCE
RegWriteM, RegWriteW
Rs1E, Rs2E, RdM, RdW
```

The final letter tells which stage the signal is in.

---

# 3. Single-Cycle vs Pipelined Processor

| Feature | Single-cycle | Pipelined |
|---|---|---|
| Instructions per cycle ideally | 1 instruction every long cycle | 1 instruction every short cycle after filling pipeline |
| CPI ideally | 1 | 1 |
| Clock cycle | long, slowest instruction path | shorter, slowest pipeline stage |
| Multiple instructions active? | no | yes |
| Main issue | slow clock | hazards |

Important concept:

> Pipelining does not make one instruction finish faster. It improves throughput by overlapping many instructions.

---

# 4. Pipeline Timing Without Hazards

For `n` instructions in a 5-stage pipeline with no stalls:

```text
Total cycles = n + 4
```

Example for 5 instructions:

```text
Cycle: 1  2  3  4  5  6  7  8  9
I1:    IF ID EX MEM WB
I2:       IF ID EX MEM WB
I3:          IF ID EX MEM WB
I4:             IF ID EX MEM WB
I5:                IF ID EX MEM WB
```

Total cycles = `5 + 4 = 9`.

---

# 5. Pipeline Hazards

A hazard is a situation that prevents the next instruction from executing normally.

Main hazards in this course:

| Hazard | Meaning |
|---|---|
| Data hazard | instruction needs a register value that is not written back yet |
| Control hazard | next PC is uncertain because of a branch/jump |

---

# 6. Data Hazards

A data hazard often occurs when one instruction writes a register and the next instruction reads it.

Example:

```text
add x3, x1, x2
sub x4, x3, x5
```

The `sub` needs `x3` before `add` has written it back.

This is a RAW dependency:

```text
Read After Write
```

---

# 7. Handling Data Hazards

Ways to handle data hazards:

1. Insert `nop` instructions.
2. Rearrange independent instructions.
3. Forward data at runtime.
4. Stall the processor at runtime.

For final exam questions, the important hardware methods are:

- forwarding
- stalling

---

# 8. Data Forwarding

Forwarding uses a result before it is written back to the register file.

The result may already be available in pipeline registers/buses.

Forwarding can send data from:

- Memory stage to Execute stage
- Writeback stage to Execute stage

## Forwarding cases

| Case | Condition | Forward select |
|---|---|---|
| Forward from Memory stage | `Rs1E == RdM` and `RegWriteM` and `Rs1E != 0` | `ForwardAE = 10` |
| Forward from Writeback stage | `Rs1E == RdW` and `RegWriteW` and `Rs1E != 0` | `ForwardAE = 01` |
| No forwarding | otherwise | `ForwardAE = 00` |

For source B, use the same idea with `Rs2E`:

```text
ForwardBE uses Rs2E instead of Rs1E
```

## Forwarding equations

```text
if ((Rs1E == RdM) AND RegWriteM AND (Rs1E != 0))
    ForwardAE = 10
else if ((Rs1E == RdW) AND RegWriteW AND (Rs1E != 0))
    ForwardAE = 01
else
    ForwardAE = 00
```

---

# 9. Load-Use Hazard

Forwarding cannot fully solve a load-use hazard when the dependent instruction immediately follows the load.

Example:

```text
lw  x5, 0(x1)
add x6, x5, x2
```

The loaded data becomes available after the memory stage, too late for the immediately following instruction's execute stage.

So one stall is needed.

## Load-use stall logic

```text
lwStall = ((Rs1D == RdE) OR (Rs2D == RdE)) AND ResultSrcE0
StallF  = lwStall
StallD  = lwStall
FlushE  = lwStall
```

Meaning:

- stall Fetch
- stall Decode
- insert bubble/flush Execute

---

# 10. Control Hazards

A control hazard happens when the processor fetches instructions before it knows whether a branch is taken.

For `beq`, the branch decision is known in the Execute stage.

If the branch is taken, instructions already fetched after the branch are wrong and must be flushed.

## Branch penalty

If branch is resolved in Execute stage:

```text
Branch misprediction/taken penalty = 2 flushed instructions
```

## Flush logic

```text
FlushD = PCSrcE
FlushE = lwStall OR PCSrcE
```

Meaning:

- if branch/jump redirects PC, clear wrong instructions in Decode and Execute pipeline registers
- if load-use stall occurs, flush Execute stage

---

# 11. Summary of Hazard Logic

## Forwarding

```text
if ((Rs1E == RdM) AND RegWriteM AND (Rs1E != 0))
    ForwardAE = 10
else if ((Rs1E == RdW) AND RegWriteW AND (Rs1E != 0))
    ForwardAE = 01
else
    ForwardAE = 00
```

Same for `ForwardBE`, replacing `Rs1E` with `Rs2E`.

## Load-use stall

```text
lwStall = ((Rs1D == RdE) OR (Rs2D == RdE)) AND ResultSrcE0
StallF = StallD = lwStall
FlushE = lwStall
```

## Control flush

```text
FlushD = PCSrcE
FlushE = lwStall OR PCSrcE
```

---

# 12. How to Draw Pipeline Timing Diagrams

Use this method every time:

1. Write each instruction in a row.
2. Put IF for instruction 1 in cycle 1.
3. Move each instruction one stage per cycle.
4. Check dependencies.
5. Add forwarding arrows where possible.
6. Add stall/bubble for load-use hazards.
7. Flush wrong-path instructions after taken branches.
8. Count final cycle of last WB.

## Basic stage pattern

```text
Instruction | C1 | C2 | C3 | C4  | C5
I1          | IF | ID | EX | MEM | WB
```

## With one stall after load-use

```text
lw  x5, 0(x1)   IF ID EX MEM WB
add x6, x5,x2      IF ID ST EX MEM WB
```

The dependent instruction stays in ID for one extra cycle, and a bubble is inserted into EX.

---

# 13. Pipeline CPI with Stalls and Flushes

Ideal pipelined CPI:

```text
CPI = 1
```

But stalls and branch penalties increase CPI.

Example style from slides:

- 25% loads
- 10% stores
- 13% branches
- 52% R-type
- 40% of loads followed by dependent instruction
- 50% of branches mispredicted/taken with penalty

Load CPI:

```text
CPI_lw = 1(0.6) + 2(0.4) = 1.4
```

Branch CPI:

```text
CPI_beq = 1(0.5) + 3(0.5) = 2
```

Average CPI:

```text
Average CPI = (0.25)(1.4) + (0.10)(1) + (0.13)(2) + (0.52)(1)
            = 1.23
```

---

# 14. Pipelined Critical Path

The pipelined cycle time is the maximum delay of all stages.

Typical formula:

```text
Tc_pipelined = max(
    Fetch stage delay,
    Decode stage delay,
    Execute stage delay,
    Memory stage delay,
    Writeback stage delay
)
```

From the lecture example, Execute stage can be critical:

```text
Tc_pipelined = tpcq + 4tmux + tALU + tAND-OR + tsetup
```

Using example values:

```text
Tc_pipelined = 40 + 4(30) + 120 + 20 + 50 = 350 ps
```

For 100 billion instructions with CPI = 1.23:

```text
Execution Time = 100 x 10^9 x 1.23 x 350 x 10^-12 = 43 seconds
```

---

# 15. Forwarding and Stall Examples

## Example 1: ALU dependency

```text
add x3, x1, x2
sub x4, x3, x5
```

The `sub` needs the result of `add`.

With forwarding:

```text
Forward from EX/MEM or MEM/WB to Execute stage of sub.
No stall needed.
```

## Example 2: Load-use dependency

```text
lw  x3, 0(x1)
add x4, x3, x5
```

Even with forwarding:

```text
One stall is needed.
```

## Example 3: Branch control hazard

```text
beq x1, x2, target
addi x3, x0, 1
add  x4, x5, x6
target:
```

If branch is taken:

```text
The two instructions after beq that were fetched on the wrong path are flushed.
```

---

# 16. Common Mistakes

| Mistake | Correction |
|---|---|
| Saying forwarding writes to register file early | Forwarding bypasses register file writeback |
| Adding stall for every dependency | ALU dependencies can often be solved by forwarding |
| Forgetting load-use stall | Immediate use of loaded value usually needs one stall |
| Flushing only one instruction for branch resolved in EX | Usually two wrong-path instructions are flushed |
| Counting pipeline cycles as `n` | Without hazards, cycles = `n + 4` for 5 stages |
| Not checking `x0` in forwarding | Do not forward from writes to x0 |

---

# 17. Likely Pipelining Exam Questions

## Question type 1: Identify read/write registers in a cycle

Example prompt:

```text
Which registers are read and written in cycle 5?
```

How to solve:

1. Fill pipeline table for cycles.
2. Find which instruction is in ID: that instruction reads registers.
3. Find which instruction is in WB: that instruction writes register.
4. Consider stalls if hazard unit is active.

## Question type 2: Draw forwarding and stalls

How to solve:

1. Find RAW dependencies.
2. For ALU result dependencies, draw forwarding to EX.
3. For `lw` immediate dependency, insert one stall.
4. Label forwarding source: MEM stage or WB stage.

## Question type 3: Pipeline with and without forwarding

Without forwarding:

- dependent instructions must wait until producer writes back.
- same-cycle register file write/read may reduce one stall.

With forwarding:

- ALU-to-ALU dependencies usually no stall.
- load-use still needs one stall.

## Question type 4: Flush for branch

How to solve:

1. Locate branch instruction.
2. Determine branch decision stage.
3. If taken, flush instructions in earlier stages.
4. Continue fetching from branch target.

## Question type 5: Pipelined performance calculation

Use:

```text
Execution Time = Instruction Count x CPI x Clock Cycle Time
Average CPI = 1 + stall/flush penalties per instruction
```

---

# 18. Exam-Focused Summary

Remember these points:

- Pipeline stages: IF, ID, EX, MEM, WB.
- Ideal pipeline CPI is 1 after filling.
- Without hazards, `n` instructions take `n + 4` cycles.
- Forwarding solves most ALU data hazards.
- Load-use hazard needs one stall.
- Taken branch resolved in EX flushes wrong-path instructions.
- Hazard unit generates forwarding, stall, and flush signals.
- Pipelining improves throughput, not individual instruction latency.
