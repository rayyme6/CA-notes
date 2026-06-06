# Computer Architecture — Final Exam Notes Roadmap

These notes turn your uploaded lectures, assignments, sessional papers, and practice questions into exam-focused study notes.

The goal is not to memorize every slide word-for-word. The goal is to understand:

- what each component does
- which control signal changes for each instruction
- how an instruction flows through the datapath
- how performance is calculated
- where stalls, forwarding, and flushing happen in a pipeline
- what kind of questions the instructor is likely to ask

---

# 1. Final Exam Priority

The final announcement says the main descriptive portion will focus on:

1. **Single-Cycle RISC-V Processor**
2. **Pipelined RISC-V Processor**

Earlier topics are still in the syllabus, but they are more likely to appear as MCQs or short numerical/conceptual questions.

A practical priority order is:

| Priority | Topic | Why it matters |
|---|---|---|
| 1 | Single-cycle datapath and control | Direct descriptive questions, control tables, dataflow diagrams |
| 2 | Pipelined datapath and hazards | Timing diagrams, forwarding, stalls, flushes, CPI |
| 3 | RISC-V instruction formats and assembly | MCQs, decoding, C to assembly, register/memory tracing |
| 4 | Performance formulas | MCQs/numericals from mid/sessional pattern |
| 5 | Arithmetic circuits | MCQs and short conceptual questions |

---

# 2. Course Map

| Chapter | Important topics | Exam use |
|---|---|---|
| Chapter 5 | Digital building blocks, adders, carry lookahead, prefix adder, ALU, flags, shifters, multiplication/division | Mostly MCQs and short conceptual questions |
| Chapter 6 | Architecture vs microarchitecture, RISC-V assembly, memory operands, branches, loops, functions, machine code, addressing modes, performance | MCQs, code tracing, small numericals |
| Chapter 7 | Performance analysis, single-cycle datapath/control, pipelined datapath/control, hazards, forwarding, stalls, flushes | Major descriptive/conceptual portion |

---

# 3. How to Study These Notes

## Step 1 — Learn the datapath story

For every instruction, ask:

> Where does the instruction start, which blocks does it use, and what does it write at the end?

Example for `lw`:

```text
PC -> Instruction Memory -> Register File -> Immediate Extend -> ALU address calculation -> Data Memory -> Register File writeback
```

Example for R-type `add`:

```text
PC -> Instruction Memory -> Register File reads rs1/rs2 -> ALU -> Register File writeback
```

## Step 2 — Memorize control by instruction type

Most datapath questions become easy if you know the instruction type:

| Type | Example | Main behavior |
|---|---|---|
| R-type | `add x3, x1, x2` | register + register, write ALU result |
| I-type ALU | `addi x3, x1, 5` | register + immediate, write ALU result |
| Load | `lw x5, 8(x3)` | address = rs1 + imm, read memory, write register |
| Store | `sw x5, 4(x3)` | address = rs1 + imm, write rs2 to memory |
| Branch | `beq x1, x2, L` | compare registers, maybe update PC |
| Jump | `jal x7, L` | write PC+4 to rd, jump to target |
| Upper immediate | `lui x6, imm` | write upper immediate to rd |

## Step 3 — Practice question formats, not just theory

Your uploaded material shows these recurring formats:

- calculate execution time, CPI, MIPS, or speedup
- convert C code to RISC-V or RISC-V to C
- trace registers, memory, or stack after every instruction/iteration
- decode machine language fields
- fill single-cycle control signals
- highlight datapath for a given instruction
- draw pipeline timing diagrams
- show forwarding, stalls, and flushes
- calculate pipeline CPI using instruction mix and penalty

---

# 4. Core Performance Formulas

```text
Execution Time = Instruction Count x CPI x Clock Cycle Time
Execution Time = (Instruction Count x CPI) / Clock Rate
CPI = Total Clock Cycles / Instruction Count
Clock Cycle Time = 1 / Clock Rate
MIPS = Clock Rate / (CPI x 10^6)
Speedup = Old Execution Time / New Execution Time
```

For weighted CPI:

```text
Average CPI = sum(Instruction fraction x CPI of that instruction class)
```

For Amdahl's Law:

```text
Speedup_overall = 1 / [(1 - f) + (f / speedup_enhanced_part)]
```

where `f` is the fraction of original execution time affected by the improvement.

---

# 5. Golden Rules for the Final

## For single-cycle questions

Use this answer structure:

1. Identify instruction type.
2. Write instruction fields used: `rs1`, `rs2`, `rd`, `imm`.
3. State datapath flow.
4. Fill control signals.
5. Mention writeback or memory effect.

## For pipelining questions

Use this answer structure:

1. Draw 5 stages: IF, ID, EX, MEM, WB.
2. Identify RAW dependencies.
3. Check if forwarding solves them.
4. For `lw` followed immediately by dependent instruction, add one stall.
5. For taken branch, flush wrong-path instructions.
6. Count total cycles.

## For machine-code questions

Use this answer structure:

1. Convert hex to binary.
2. Read opcode bits `[6:0]` first.
3. Use opcode to identify format.
4. Extract `rd`, `rs1`, `rs2`, `funct3`, `funct7`, and immediate.
5. Use opcode + funct fields to determine instruction.

---

# 6. Exam-Focused Summary

Remember these points:

- **Architecture** is the programmer-visible instruction set and operand locations.
- **Microarchitecture** is the hardware implementation of the architecture.
- A processor has a **datapath** and a **control unit**.
- In a **single-cycle processor**, every instruction completes in one long clock cycle.
- In a **pipelined processor**, multiple instructions overlap across stages.
- Pipelining improves throughput, not the latency of one instruction.
- Data hazards happen when a needed register value has not been written back yet.
- Forwarding solves many ALU hazards.
- Load-use hazards usually need one stall even with forwarding.
- Control hazards from taken branches require flushing wrong-path instructions.
