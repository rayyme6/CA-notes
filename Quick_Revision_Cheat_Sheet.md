# Computer Architecture Final — Quick Revision Cheat Sheet

## 1. Performance

```text
Execution Time = Instruction Count x CPI x Clock Cycle Time
Execution Time = Instruction Count x CPI / Clock Rate
CPI = Total Cycles / Instruction Count
Clock Cycle Time = 1 / Clock Rate
Speedup = Old Time / New Time
Average CPI = sum(fraction x CPI)
```

---

# 2. RISC-V Instruction Types

| Type | Example | Fields |
|---|---|---|
| R | `add x3,x1,x2` | rd, rs1, rs2 |
| I | `addi x3,x1,5`, `lw x5,8(x3)` | rd, rs1, imm |
| S | `sw x5,8(x3)` | rs1, rs2, imm |
| B | `beq x1,x2,L` | rs1, rs2, imm |
| U | `lui x6,1000` | rd, imm |
| J | `jal x7,L` | rd, imm |

---

# 3. Important Opcodes

| Instruction | Opcode decimal | Type |
|---|---:|---|
| R-type | 51 | R |
| addi | 19 | I |
| lw | 3 | I |
| sw | 35 | S |
| beq/bne | 99 | B |
| jal | 111 | J |
| jalr | 103 | I |
| lui | 55 | U |

---

# 4. Single-Cycle Control Signals

| Instruction | RegWrite | MemRead | MemWrite | MemtoReg | ALUSrc | Branch | ALUOp | ResultSrc | ImmSrc |
|---|---:|---:|---:|---|---:|---:|---|---|---|
| R-type | 1 | 0 | 0 | 0 | 0 | 0 | 10 | 00 | XX |
| I-type ALU | 1 | 0 | 0 | 0 | 1 | 0 | 10 | 00 | 00 |
| `lw` | 1 | 1 | 0 | 1 | 1 | 0 | 00 | 01 | 00 |
| `sw` | 0 | 0 | 1 | X | 1 | 0 | 00 | XX | 01 |
| `beq/bne` | 0 | 0 | 0 | X | 0 | 1 | 01 | XX | 10 |
| `jal` | 1 | 0 | 0 | 0 | X | 0 | XX | 10 | 11 |
| `lui` | 1 | 0 | 0 | 0 | X | 0 | XX/00 | ImmExt | U |

---

# 5. ALU Control

| ALUControl | Function |
|---|---|
| 000 | add |
| 001 | subtract |
| 010 | and |
| 011 | or |
| 101 | slt |

---

# 6. Immediate Source

| ImmSrc | Type |
|---|---|
| 00 | I-type |
| 01 | S-type |
| 10 | B-type |
| 11 | J-type |
| U | U-type for `lui`, if added |

---

# 7. Datapath Flows

## R-type

```text
PC -> IM -> RF(rs1,rs2) -> ALU -> RF(rd)
```

## Load

```text
PC -> IM -> RF(rs1) + Imm -> ALU address -> Data Memory read -> RF(rd)
```

## Store

```text
PC -> IM -> RF(rs1,rs2) + Imm -> ALU address -> Data Memory write
```

## Branch

```text
PC -> IM -> RF(rs1,rs2) -> ALU subtract -> Zero -> PCSrc -> PC target
```

## JAL

```text
PC+4 -> rd
PC + J-imm -> PCNext
```

---

# 8. Pipeline Basics

Stages:

```text
IF -> ID -> EX -> MEM -> WB
```

No hazards for `n` instructions:

```text
cycles = n + 4
```

---

# 9. Forwarding

```text
if ((Rs1E == RdM) AND RegWriteM AND (Rs1E != 0))
    ForwardAE = 10
else if ((Rs1E == RdW) AND RegWriteW AND (Rs1E != 0))
    ForwardAE = 01
else
    ForwardAE = 00
```

Same for B source using `Rs2E`.

---

# 10. Load-Use Stall

```text
lwStall = ((Rs1D == RdE) OR (Rs2D == RdE)) AND ResultSrcE0
StallF = StallD = lwStall
FlushE = lwStall
```

Meaning:

```text
stall Fetch and Decode, flush Execute
```

---

# 11. Branch Flush

```text
FlushD = PCSrcE
FlushE = lwStall OR PCSrcE
```

If branch resolved in EX and taken:

```text
flush 2 wrong-path instructions
```

---

# 12. Chapter 5 MCQ Facts

- Ripple-carry adder: simple but slow.
- Carry-lookahead adder: uses generate and propagate.
- Prefix adder: faster carry computation using tree structure.
- `Gi = AiBi`.
- `Pi = Ai + Bi`.
- `Ci = Gi + PiCi-1`.
- ALU flags: N, Z, C, V.
- Arithmetic right shift preserves sign bit.
- `div` is signed; `divu` is unsigned.

---

# 13. Most Important Final Lines

Memorize these:

```text
Single-cycle: CPI = 1, long clock cycle, lw critical path.
Pipeline: ideal CPI = 1, but hazards add stalls/flushes.
Forwarding solves ALU RAW hazards.
Load-use hazard needs one stall.
Taken branch in EX flushes two instructions.
Control signals depend mainly on instruction type.
```
