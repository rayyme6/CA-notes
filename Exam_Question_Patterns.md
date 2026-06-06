# Final Exam — How Questions Are Likely to Be Asked

## 1. Big Picture

Based on the uploaded syllabus, assignments, sessional papers, datapath worksheets, and pipeline practice file, the final is likely to mix:

- MCQs from earlier topics
- conceptual/descriptive questions from single-cycle and pipelined RISC-V processors
- control-signal tables
- datapath/dataflow questions
- pipeline timing/hazard questions
- small performance numericals

The highest-yield areas are:

```text
Single-Cycle RISC-V Processor
Pipelined RISC-V Processor
Datapath + Control
Hazards + Forwarding + Stalls + Flushes
```

---

# 2. Likely Question Pattern 1 — Single-Cycle Control Table

## What they may ask

They may give a short RISC-V program and ask:

```text
Fill the datapath control signals table for each instruction and mention instruction type.
```

This matches your uploaded Section E/F style.

## Example instruction list

```text
addi x1, x0, 5
add  x3, x1, x2
sub  x4, x3, x1
lw   x5, 0(x3)
sw   x5, 4(x3)
beq  x1, x2, LABEL
lui  x6, 1000
jal  x7, LABEL
```

## How to answer

Use this control table:

| Instruction | Type | RegWrite | MemRead | MemWrite | MemtoReg | ALUSrc | Branch | ALUOp | ResultSrc | ImmSrc |
|---|---|---:|---:|---:|---|---:|---:|---|---|---|
| `add/sub/and/or` | R | 1 | 0 | 0 | 0 | 0 | 0 | 10 | 00 | XX |
| `addi/andi/ori` | I | 1 | 0 | 0 | 0 | 1 | 0 | 10 | 00 | 00 |
| `lw` | I/load | 1 | 1 | 0 | 1 | 1 | 0 | 00 | 01 | 00 |
| `sw` | S | 0 | 0 | 1 | X | 1 | 0 | 00 | XX | 01 |
| `beq/bne` | B | 0 | 0 | 0 | X | 0 | 1 | 01 | XX | 10 |
| `jal` | J | 1 | 0 | 0 | 0 | X | 0 | XX | 10 | 11 |
| `lui` | U | 1 | 0 | 0 | 0 | X | 0 | XX/00 | ImmExt path | U |

## Marks-saving tip

If `lui` is included, mention that the datapath must have U-type immediate extension and a path to write the immediate to `rd`.

---

# 3. Likely Question Pattern 2 — Highlight Datapath/Dataflow

## What they may ask

They may give one instruction and ask you to:

```text
Fill control signals and highlight dataflow on the datapath diagram.
```

This matches Assignment 3.

## How to answer for common instructions

### R-type `add s1, s5, s3`

Active path:

```text
PC -> Instruction Memory -> Register File reads s5 and s3 -> ALU add -> Result mux selects ALUResult -> Register File writes s1 -> PC+4
```

Control:

```text
RegWrite=1, ALUSrc=0, MemWrite=0, ResultSrc=00, Branch=0, ALUOp=10
```

### I-type `andi s1, s5, -5`

Active path:

```text
PC -> Instruction Memory -> Register File reads s5 -> Extend immediate -> ALU AND -> Result mux selects ALUResult -> Register File writes s1 -> PC+4
```

Control:

```text
RegWrite=1, ImmSrc=00, ALUSrc=1, MemWrite=0, ResultSrc=00, Branch=0, ALUOp=10
```

### Load `lw s1, 8(s5)`

Active path:

```text
Register File reads base s5 -> Extend immediate -> ALU address -> Data Memory read -> write ReadData to s1
```

Control:

```text
RegWrite=1, MemRead=1, MemWrite=0, MemtoReg=1, ALUSrc=1, ImmSrc=00, ALUOp=00
```

### Store `sw s1, 8(s5)`

Active path:

```text
Register File reads base s5 and data s1 -> Extend S-type immediate -> ALU address -> Data Memory write
```

Control:

```text
RegWrite=0, MemWrite=1, ALUSrc=1, ImmSrc=01, ALUOp=00
```

### Branch `beq s1, s2, label`

Active path:

```text
Register File reads s1 and s2 -> ALU subtract -> Zero flag -> PCSrc selects branch target if taken
```

Control:

```text
RegWrite=0, MemWrite=0, ALUSrc=0, Branch=1, ImmSrc=10, ALUOp=01
```

---

# 4. Likely Question Pattern 3 — Modify Datapath

## What they may ask

They may ask you to modify the datapath for an instruction not fully supported.

Examples:

- `lui`
- `bne`
- shift instruction
- comparator / `slt`
- `jal`

## How to answer

Use this structure:

1. State what current datapath cannot do.
2. Add required block/mux/control signal.
3. Explain dataflow for the new instruction.
4. Fill control table.

## Example: Add `lui`

Problem:

```text
lui writes {imm[31:12], 12'b0} to rd.
```

Needed change:

- U-type immediate extension.
- Add ImmExt as possible input to Result mux.
- Add `ResultSrc` code for immediate writeback.

Control idea:

```text
RegWrite=1, ImmSrc=U-type, ResultSrc=ImmExt, MemWrite=0, Branch=0
```

## Example: Add `bne`

Problem:

```text
beq branches when Zero=1, but bne branches when Zero=0.
```

Needed change:

- Add branch-condition control.
- Use inverted Zero for `bne`.

Control idea:

```text
PCSrc = Branch AND NOT Zero
```

---

# 5. Likely Question Pattern 4 — Pipeline Timing Diagram

## What they may ask

They may give RISC-V code and ask:

```text
Show the timing diagram for a 5-stage pipeline.
```

or:

```text
Show forwarding and stalls.
```

## How to answer

Draw this table:

| Instruction | C1 | C2 | C3 | C4 | C5 | C6 | C7 | C8 | C9 |
|---|---|---|---|---|---|---|---|---|---|
| I1 | IF | ID | EX | MEM | WB |  |  |  |  |
| I2 |  | IF | ID | EX | MEM | WB |  |  |  |
| I3 |  |  | IF | ID | EX | MEM | WB |  |  |

Then adjust for stalls and flushes.

## Key rules

| Situation | Action |
|---|---|
| ALU result used by next instruction | forwarding, usually no stall |
| `lw` result used immediately next | one stall |
| taken branch resolved in EX | flush wrong-path instructions |
| no hazards for `n` instructions | cycles = `n + 4` |

---

# 6. Likely Question Pattern 5 — Forwarding / Stalling / Flushing

## What they may ask

They may ask:

```text
Show forwarding and stalls needed to execute this code.
```

## How to identify hazards

Search for RAW dependencies:

```text
producer writes rd
consumer later reads same register as rs1/rs2
```

Example:

```text
add x3, x1, x2
sub x4, x3, x5
```

`sub` reads `x3`, which `add` writes.

## Forwarding rules

Use forwarding when source register in Execute stage matches destination register in Memory or Writeback stage.

```text
Forward from M stage if RsE == RdM and RegWriteM
Forward from W stage if RsE == RdW and RegWriteW
```

## Load-use stall rule

```text
lw x5, 0(x1)
add x6, x5, x2
```

This needs one stall.

## Branch flush rule

If branch taken in EX:

```text
flush Decode and Execute wrong-path instructions
```

---

# 7. Likely Question Pattern 6 — Registers Read/Written in a Given Cycle

## What they may ask

A practice question asks something like:

```text
Which registers are being written and which are being read on the fifth cycle?
```

## How to answer

1. Draw pipeline table up to cycle 5.
2. The instruction in ID reads registers.
3. The instruction in WB writes register.
4. If a stage has a stall/bubble, adjust the table.

Example no-stall pipeline:

| Cycle 5 stage | Instruction |
|---|---|
| WB | I1 writes destination register |
| MEM | I2 uses memory if needed |
| EX | I3 uses ALU operands |
| ID | I4 reads source registers |
| IF | I5 fetches instruction |

---

# 8. Likely Question Pattern 7 — Performance Numerical

## What they may ask

They may give instruction count, CPI, and clock rate and ask execution time.

Use:

```text
Execution Time = Instruction Count x CPI / Clock Rate
```

## Weighted CPI

If instruction classes are given:

```text
Average CPI = sum(fraction x CPI)
```

## Pipeline CPI

If stalls/flushes are given:

```text
Average CPI = 1 + average stall cycles per instruction
```

or use weighted CPI by instruction class.

---

# 9. Likely Question Pattern 8 — RISC-V Code Tracing

## What they may ask

They may ask you to trace:

- registers after each instruction
- memory after each loop iteration
- stack changes during function calls
- signed/unsigned multiplication/division results

## How to answer

Use this table:

| Step | Instruction | Affected register/memory | Decimal value | Hex value |
|---|---|---|---:|---|
| 1 | `addi s0, zero, 25` | s0 | 25 | 0x00000019 |
| 2 | `addi s1, zero, 2` | s1 | 2 | 0x00000002 |

For signed vs unsigned:

- signed interpretation uses two's complement
- unsigned interpretation treats all bits as positive

---

# 10. Likely Question Pattern 9 — C to RISC-V and RISC-V to C

## What they may ask

They may give:

- if/else
- while loop
- for loop
- arrays
- function calls

## C to assembly template

```text
initialize variables
loop_label:
    branch_if_condition_false done
    body
    update
    j loop_label
done:
```

## RISC-V to C method

1. Identify variables from register comments.
2. Find loop labels and branch exits.
3. Translate branches into `if`, `while`, or `for`.
4. Translate arithmetic and memory operations.

---

# 11. Likely MCQs from Earlier Topics

## Chapter 5 MCQs

- Ripple-carry adder is slow because carry ripples through all stages.
- Carry-lookahead uses generate and propagate.
- ALU flags include N, Z, C, V.
- Arithmetic right shift preserves sign bit.
- `div` is signed; `divu` is unsigned.

## Chapter 6 MCQs

- Architecture is programmer-visible; microarchitecture is hardware implementation.
- Assembly is human-readable; machine language is binary.
- RISC-V has 32 general-purpose registers.
- `x0` is constant zero.
- `lw` uses base addressing.
- Branches and jumps use PC-relative addressing.
- `jal` stores `PC+4` in destination register.
- Machine-code decoding starts with opcode.

## Chapter 7 MCQs

- Single-cycle CPI is 1, but clock cycle is long.
- `lw` usually creates the single-cycle critical path.
- Pipeline stages are IF, ID, EX, MEM, WB.
- Forwarding solves many RAW hazards.
- Load-use hazard requires one stall.
- Taken branch in EX flushes two wrong-path instructions.

---

# 12. Short Descriptive Questions They May Ask

## Single-cycle

1. Explain the execution of `lw` in a single-cycle datapath.
2. Explain the control signals for R-type instruction.
3. Why is `lw` the critical path in a single-cycle processor?
4. How does `beq` update the PC?
5. What modifications are needed for `jal` or `lui`?

## Pipelined

1. Explain the five stages of a RISC-V pipeline.
2. What is a data hazard? Give an example.
3. How does forwarding solve a data hazard?
4. Why does a load-use hazard still require a stall?
5. What is a control hazard and how is it handled?
6. Explain flush vs stall.
7. Calculate average CPI with load stalls and branch flush penalties.

---

# 13. Last-Day Strategy

Do these in order:

1. Memorize the single-cycle control table.
2. Practice writing dataflow for `lw`, `sw`, R-type, `beq`, `jal`, and `lui`.
3. Practice one pipeline timing diagram with forwarding.
4. Practice one pipeline timing diagram with load-use stall.
5. Practice branch flushing.
6. Revise performance formulas.
7. Revise RISC-V instruction formats and opcodes.
8. Revise Chapter 5 MCQ facts.

---

# 14. One-Page Answer Templates

## Template: Control table answer

```text
Instruction: __________
Type: __________
RegWrite: ___
MemRead: ___
MemWrite: ___
MemtoReg/ResultSrc: ___
ALUSrc: ___
Branch: ___
ALUOp: ___
ImmSrc: ___
Dataflow: __________
```

## Template: Pipeline hazard answer

```text
Dependency: Instruction ___ writes x__, Instruction ___ reads x__.
Hazard type: RAW data hazard.
Solution: forwarding from ___ stage to EX stage / one stall if load-use.
If branch taken: flush ___ wrong-path instructions.
Total cycles: ___
```

## Template: Performance answer

```text
Given:
Instruction count = ___
CPI = ___
Clock rate = ___

Formula:
Execution Time = IC x CPI / Clock Rate

Substitution:
Execution Time = ___
```
