# Chapter 7 — Single-Cycle RISC-V Processor

## 1. Main Objective

This is one of the most important final-exam topics.

The main idea is:

> A single-cycle processor executes each instruction completely in one clock cycle.

The cycle must be long enough for the slowest instruction, usually `lw`, to complete.

---

# 2. Microarchitecture Basics

A processor has two major parts:

| Part | Meaning |
|---|---|
| Datapath | hardware blocks that store, move, and process data |
| Control | signals that tell datapath blocks what to do |

Examples of datapath blocks:

- PC
- instruction memory
- register file
- immediate extension unit
- ALU
- data memory
- multiplexers
- adders

Examples of control signals:

- `RegWrite`
- `ImmSrc`
- `ALUSrc`
- `MemWrite`
- `ResultSrc`
- `Branch`
- `ALUOp`
- `Jump`

---

# 3. RISC-V Subset Used in Datapath

The single-cycle datapath usually starts with this subset:

| Category | Instructions |
|---|---|
| R-type ALU | `add`, `sub`, `and`, `or`, `slt` |
| Memory | `lw`, `sw` |
| Branch | `beq` |

Then it is extended to include:

| Extension | Examples |
|---|---|
| I-type ALU | `addi`, `andi`, `ori`, `slti` |
| Jump | `jal` |
| Upper immediate | `lui` |
| Branch not equal | `bne` |

---

# 4. Common Instruction Execution Flows

## `lw rd, imm(rs1)`

Example:

```text
lw x6, -4(x9)
```

Steps:

1. Fetch instruction from instruction memory using PC.
2. Read base register `rs1` from register file.
3. Sign-extend immediate.
4. ALU computes address: `rs1 + imm`.
5. Data memory reads from that address.
6. Write memory data back to `rd`.
7. PC becomes `PC + 4`.

Datapath flow:

```text
PC -> Instruction Memory -> Register File(rs1)
Instruction immediate -> Extend
rs1 + ImmExt -> ALUResult address
Data Memory ReadData -> Result mux -> rd
PC -> PC+4 -> PCNext
```

## `sw rs2, imm(rs1)`

Steps:

1. Fetch instruction.
2. Read `rs1` for base address.
3. Read `rs2` for data to store.
4. Sign-extend S-type immediate.
5. ALU computes address: `rs1 + imm`.
6. Data memory writes `rs2` value at computed address.
7. PC becomes `PC + 4`.

## R-type instruction

Example:

```text
add x3, x1, x2
```

Steps:

1. Fetch instruction.
2. Read `rs1` and `rs2` from register file.
3. ALU performs operation.
4. Write ALU result to `rd`.
5. PC becomes `PC + 4`.

## `beq rs1, rs2, label`

Steps:

1. Fetch instruction.
2. Read `rs1` and `rs2`.
3. Extend B-type immediate.
4. ALU subtracts `rs1 - rs2`.
5. If zero flag is 1, branch is taken.
6. Branch target = `PC + ImmExt`.
7. PC selects branch target if taken, otherwise `PC + 4`.

## `jal rd, label`

Steps:

1. Fetch instruction.
2. Extend J-type immediate.
3. Compute target address: `PC + ImmExt`.
4. Write `PC + 4` to `rd`.
5. PC jumps to target.

---

# 5. Immediate Extension (`ImmSrc`)

The immediate format depends on instruction type.

| ImmSrc | Immediate format | Instruction type |
|---|---|---|
| `00` | `{{20{instr[31]}}, instr[31:20]}` | I-type |
| `01` | `{{20{instr[31]}}, instr[31:25], instr[11:7]}` | S-type |
| `10` | `{{19{instr[31]}}, instr[31], instr[7], instr[30:25], instr[11:8], 1'b0}` | B-type |
| `11` | `{{12{instr[31]}}, instr[19:12], instr[20], instr[30:21], 1'b0}` | J-type |

For `lui`, many datapaths add a U-type immediate path:

```text
ImmExt = {instr[31:12], 12'b0}
```

If your paper includes `lui`, write that the datapath must support U-type immediate and select it for writeback.

---

# 6. Main Decoder Control Table

This table is the heart of single-cycle control.

| Instruction | op | RegWrite | ImmSrc | ALUSrc | MemWrite | ResultSrc | Branch | ALUOp | Jump |
|---|---:|---:|---|---:|---:|---|---:|---|---:|
| `lw` | 3 | 1 | 00 | 1 | 0 | 01 | 0 | 00 | 0 |
| `sw` | 35 | 0 | 01 | 1 | 1 | XX | 0 | 00 | 0 |
| R-type | 51 | 1 | XX | 0 | 0 | 00 | 0 | 10 | 0 |
| `beq` | 99 | 0 | 10 | 0 | 0 | XX | 1 | 01 | 0 |
| I-type ALU | 19 | 1 | 00 | 1 | 0 | 00 | 0 | 10 | 0 |
| `jal` | 111 | 1 | 11 | X | 0 | 10 | 0 | XX | 1 |

## Meaning of main signals

| Signal | Meaning |
|---|---|
| `RegWrite` | write result into register file |
| `ImmSrc` | choose immediate format |
| `ALUSrc` | ALU second input comes from immediate instead of register `rs2` |
| `MemWrite` | write to data memory |
| `ResultSrc` | choose what writes back to register file |
| `Branch` | instruction is conditional branch |
| `ALUOp` | high-level ALU instruction class |
| `Jump` | instruction is an unconditional jump |

---

# 7. ResultSrc Values

Typical `ResultSrc` values:

| ResultSrc | Writeback source |
|---|---|
| `00` | ALUResult |
| `01` | ReadData from Data Memory |
| `10` | PCPlus4 |
| `11` | ImmExt / U-type immediate, if extended for `lui` |

Older simplified tables may use `MemtoReg` instead:

| MemtoReg | Meaning |
|---|---|
| 0 | write ALU result |
| 1 | write memory read data |
| X | don't care |

---

# 8. ALU Decoder

The main decoder gives `ALUOp`. The ALU decoder uses `ALUOp`, `funct3`, and `funct7` to generate `ALUControl`.

| ALUOp | funct3 | funct7/op bit pattern | Instruction | ALUControl |
|---|---|---|---|---|
| 00 | X | X | `lw`, `sw` | `000` add |
| 01 | X | X | `beq` | `001` subtract |
| 10 | 000 | add pattern | `add` / `addi` | `000` add |
| 10 | 000 | sub pattern | `sub` | `001` subtract |
| 10 | 010 | X | `slt` / `slti` | `101` set less than |
| 10 | 110 | X | `or` / `ori` | `011` or |
| 10 | 111 | X | `and` / `andi` | `010` and |

---

# 9. Master Control Table for Common Exam Questions

This table matches the style of the Section E/F datapath-control questions.

| Instruction type | RegWrite | MemRead | MemWrite | MemtoReg | ALUSrc | Branch | ALUOp | ResultSrc | ImmSrc |
|---|---:|---:|---:|---|---:|---:|---|---|---|
| R-type `add/sub/and/or` | 1 | 0 | 0 | 0 | 0 | 0 | 10 | 00 | XX |
| I-type ALU `addi/andi/ori` | 1 | 0 | 0 | 0 | 1 | 0 | 10 | 00 | 00 |
| `lw` | 1 | 1 | 0 | 1 | 1 | 0 | 00 | 01 | 00 |
| `sw` | 0 | 0 | 1 | X | 1 | 0 | 00 | XX | 01 |
| `beq` | 0 | 0 | 0 | X | 0 | 1 | 01 | XX | 10 |
| `bne` | 0 | 0 | 0 | X | 0 | 1 | 01 | XX | 10 |
| `jal` | 1 | 0 | 0 | 0 | X | 0 | XX | 10 | 11 |
| `lui` | 1 | 0 | 0 | 0 | X/1 | 0 | XX/00 | 11 or ImmExt path | U-type |

Important note for `bne`:

- `beq` branches when `Zero = 1`.
- `bne` branches when `Zero = 0`.
- If the datapath only supports `beq`, you must add/invert branch condition logic for `bne`.

Important note for `lui`:

- If no U-type writeback path exists, the datapath must be modified.
- A common modification is to add an immediate path to the writeback mux.

---

# 10. PCSrc Logic

For `beq`:

```text
PCSrc = Branch AND Zero
```

For `bne`:

```text
PCSrc = Branch AND NOT Zero
```

For `jal`:

```text
PCSrc = 1
```

For combined branch/jump control:

```text
PCSrc = (Branch AND condition_true) OR Jump
```

---

# 11. Single-Cycle Performance

Program execution time:

```text
Execution Time = Instruction Count x CPI x Clock Cycle Time
```

For single-cycle processor:

```text
CPI = 1
```

But the clock cycle is long because it must fit the slowest instruction.

## Critical path

The critical path is usually `lw`:

```text
PC -> Instruction Memory -> Register File -> ALU -> Data Memory -> Writeback Mux -> Register File
```

Typical formula from slides:

```text
Tc_single = tpcq_PC + 2tmem + tRFread + tALU + tmux + tRFsetup
```

Using example delay values:

```text
Tc_single = 40 + 2(200) + 100 + 120 + 30 + 60 = 750 ps
```

For 100 billion instructions:

```text
Execution Time = 100 x 10^9 x 1 x 750 x 10^-12 = 75 seconds
```

---

# 12. How to Answer a Datapath Highlighting Question

Use this template:

## For `add rd, rs1, rs2`

```text
Instruction Memory -> Register File reads rs1 and rs2 -> ALU performs add -> Result mux selects ALUResult -> Register File writes rd -> PCPlus4 selected for next PC
```

Control:

```text
RegWrite=1, ALUSrc=0, MemWrite=0, ResultSrc=00, Branch=0, ALUOp=10
```

## For `lw rd, imm(rs1)`

```text
Instruction Memory -> Register File reads rs1 -> Extend immediate -> ALU computes address -> Data Memory reads -> Result mux selects ReadData -> Register File writes rd -> PCPlus4 selected
```

Control:

```text
RegWrite=1, ImmSrc=00, ALUSrc=1, MemWrite=0, ResultSrc=01, Branch=0, ALUOp=00
```

## For `sw rs2, imm(rs1)`

```text
Instruction Memory -> Register File reads rs1 and rs2 -> Extend S-type immediate -> ALU computes address -> Data Memory writes rs2 value -> no register write
```

Control:

```text
RegWrite=0, ImmSrc=01, ALUSrc=1, MemWrite=1, Branch=0, ALUOp=00
```

## For `beq rs1, rs2, label`

```text
Instruction Memory -> Register File reads rs1 and rs2 -> Extend B-type immediate -> ALU subtracts rs1-rs2 -> Zero determines PCSrc -> PC selects target or PC+4
```

Control:

```text
RegWrite=0, ImmSrc=10, ALUSrc=0, MemWrite=0, Branch=1, ALUOp=01
```

---

# 13. Common Mistakes

| Mistake | Correction |
|---|---|
| Setting `RegWrite=1` for `sw` | Store writes memory, not register file |
| Setting `MemWrite=1` for `lw` | Load reads memory, not writes |
| Using I-type immediate for `sw` | Store uses S-type immediate |
| Forgetting `jal` writes `PC+4` to rd | `jal` is both jump and link |
| Treating `bne` same as `beq` condition | `bne` branches when Zero is 0 |
| Forgetting `lui` needs U-type immediate | `lui` writes upper immediate, not ALU register result |

---

# 14. Likely Single-Cycle Exam Questions

## Question type 1: Fill control signal table

Example prompt:

```text
addi x1, x0, 5
lw x5, 0(x3)
sw x5, 4(x3)
beq x1, x2, LABEL
jal x7, LABEL
lui x6, 1000
```

What to do:

1. Identify each instruction type.
2. Fill signals from the master control table.
3. Mark don't-cares as `X` or `XX` if allowed.

## Question type 2: Highlight datapath for instruction

What to do:

1. Show active blocks.
2. Show active mux selections.
3. Mention register/memory write.
4. Mention PC selection.

## Question type 3: Modify datapath

Likely modifications:

- add `bne` support by adding branch-not-equal logic
- add `lui` support by adding U-type immediate and writeback mux input
- add shift/compare operation support in ALU
- add `jal` support by adding PC+4 writeback path and J-type immediate

---

# 15. Exam-Focused Summary

Remember these points:

- Single-cycle processor has CPI = 1.
- The clock cycle time is limited by the slowest instruction.
- `lw` usually creates the critical path.
- Datapath tells where data goes.
- Control tells which path is active.
- For most questions, first identify instruction type, then fill control signals.
- `ResultSrc` selects what is written back.
- `ALUSrc` selects register vs immediate for ALU second input.
- `ImmSrc` selects immediate format.
- `ALUOp` tells the ALU decoder what class of operation is needed.
