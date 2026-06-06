# Computer Architecture — Combined Final Exam Notes

This file combines the generated notes in one place.


---

<!-- Source file: BASE.md -->

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


---

<!-- Source file: Chapter_05_Arithmetic_Circuits.md -->

# Chapter 5 — Arithmetic Circuits and Digital Building Blocks

## 1. Main Objective

This chapter explains the hardware building blocks used later to build the processor datapath.

The important idea is:

> A processor is built from smaller reusable digital blocks such as adders, multiplexers, registers, ALUs, memories, and control logic.

For the final, Chapter 5 is likely to appear mostly as MCQs or short conceptual questions.

---

# 2. Digital Building Blocks

Digital building blocks include:

- gates
- multiplexers
- decoders
- registers
- arithmetic circuits
- counters
- memory arrays
- logic arrays

They demonstrate three design ideas:

| Idea | Meaning |
|---|---|
| Hierarchy | build complex systems from simple parts |
| Modularity | each part has a clear function and interface |
| Regularity | same structure can be repeated for many bits |

These blocks are used in Chapter 7 to build the RISC-V processor.

---

# 3. 1-Bit Adders

## Half adder

A half adder adds two 1-bit inputs.

```text
Inputs:  A, B
Outputs: Sum, Carry

Sum   = A XOR B
Carry = A AND B
```

## Full adder

A full adder adds two input bits plus a carry-in.

```text
Inputs:  A, B, Cin
Outputs: Sum, Cout

Sum  = A XOR B XOR Cin
Cout = AB + ACin + BCin
```

A full adder is the basic cell used in multi-bit adders.

---

# 4. Ripple-Carry Adder

A ripple-carry adder chains 1-bit full adders together.

```text
FA0 -> FA1 -> FA2 -> ... -> FA(N-1)
```

The carry out from one full adder becomes the carry in of the next full adder.

## Key point

> Ripple-carry adders are simple but slow because the carry must ripple through the entire chain.

Delay:

```text
tripple = N x tFA
```

where:

- `N` = number of bits
- `tFA` = delay of one full adder

## MCQ clue

If the question says **slow but simple**, the answer is usually **ripple-carry adder**.

---

# 5. Carry-Lookahead Adder

Carry-lookahead adders reduce delay by computing carry signals using generate and propagate logic.

For bit column `i`:

```text
Generate:  Gi = Ai Bi
Propagate: Pi = Ai + Bi
Carry out: Ci = Gi + Pi Ci-1
```

## Meaning of generate and propagate

| Signal | Meaning |
|---|---|
| Generate `Gi` | this bit position creates a carry by itself |
| Propagate `Pi` | this bit position passes an incoming carry forward |

## 4-bit block propagate

```text
P3:0 = P3 P2 P1 P0
```

A block propagates a carry only if all four bit positions propagate it.

## 4-bit block generate

```text
G3:0 = G3 + G2P3 + G1P2P3 + G0P1P2P3
```

Meaning:

- bit 3 generates a carry, or
- bit 2 generates and bit 3 propagates, or
- bit 1 generates and bits 2-3 propagate, or
- bit 0 generates and bits 1-3 propagate.

## Exam-focused summary

- Ripple carry waits for carries one after another.
- Carry lookahead predicts carries earlier.
- Carry lookahead is faster for large adders but uses more hardware.

---

# 6. Prefix Adders

A prefix adder is another fast carry-propagate adder.

The main idea is:

> Compute carries in parallel using a tree-like structure of generate/propagate combinations.

Comparison:

| Adder | Speed | Hardware cost | Concept |
|---|---|---|---|
| Ripple-carry | slow | low | carry ripples through all bits |
| Carry-lookahead | fast | medium/high | block generate/propagate logic |
| Prefix | faster | high | tree-based parallel carry computation |

---

# 7. ALU — Arithmetic Logic Unit

The ALU performs arithmetic and logical operations.

Common operations:

| Operation | Meaning |
|---|---|
| add | addition |
| sub | subtraction |
| and | bitwise AND |
| or | bitwise OR |
| slt | set less than |

For the Chapter 7 processor, the ALU control table is usually:

| ALUControl | Function |
|---|---|
| `000` | add |
| `001` | subtract |
| `010` | and |
| `011` | or |
| `101` | slt |

---

# 8. Subtraction Using an Adder

Hardware usually performs subtraction using two's complement.

```text
A - B = A + (~B + 1)
```

So the same adder can be reused for both addition and subtraction.

Typical method:

- invert `B`
- set carry-in to 1
- add using the same adder

---

# 9. ALU Status Flags

ALUs often produce flags.

| Flag | Meaning |
|---|---|
| N | Negative result: most significant bit of result is 1 |
| Z | Zero result: all result bits are 0 |
| C | Carry out from addition/subtraction |
| V | Signed overflow occurred |

## Zero flag

```text
Z = 1 if result == 0
```

This is important for branches like `beq`, because the ALU subtracts two values and checks whether the result is zero.

## Overflow flag

Overflow matters for signed arithmetic.

For addition:

- adding two positives gives a negative -> overflow
- adding two negatives gives a positive -> overflow

For subtraction:

- signs differ and result sign is wrong -> overflow

---

# 10. Signed vs Unsigned Comparison

A common MCQ trap is mixing signed and unsigned comparisons.

| Comparison | Uses |
|---|---|
| signed less-than | sign and overflow matter |
| unsigned less-than | carry/borrow matters |

In RISC-V:

- `slt` is signed set-less-than
- `sltu` is unsigned set-less-than

Example idea:

```text
0xFFFFFFFF is -1 in signed interpretation
0xFFFFFFFF is 4294967295 in unsigned interpretation
```

So the same bits can compare differently depending on signed/unsigned interpretation.

---

# 11. Shifters

Shift operations move bits left or right.

| Instruction idea | Meaning |
|---|---|
| logical left shift | fills right side with 0s |
| logical right shift | fills left side with 0s |
| arithmetic right shift | fills left side with old sign bit |

## Shifts as multiplication/division

For unsigned or positive binary numbers:

```text
shift left by k  = multiply by 2^k
shift right by k = divide by 2^k
```

Example:

```text
x << 3 = x x 8
x >> 2 = x / 4
```

## MCQ clue

If the value is signed and negative, arithmetic right shift preserves the sign bit; logical right shift does not.

---

# 12. Multiplication and Division Hardware

## Multiplication

A `32 x 32` multiplication produces a 64-bit result.

RISC-V examples:

```text
mul  rd, rs1, rs2   # lower 32 bits
mulh rd, rs1, rs2   # upper 32 bits, signed
```

## Division

Division produces quotient and remainder.

```text
div rd, rs1, rs2   # signed quotient
rem rd, rs1, rs2   # signed remainder
divu rd, rs1, rs2  # unsigned quotient
remu rd, rs1, rs2  # unsigned remainder
```

Common exam trick:

- `div` and `rem` treat operands as signed.
- `divu` and `remu` treat the same bit patterns as unsigned.

---

# 13. Fixed-Point vs Floating-Point

| Type | Main idea | Best for |
|---|---|---|
| Fixed-point | binary point stays fixed | hardware efficiency, signal processing |
| Floating-point | binary point moves like scientific notation | general-purpose programming, large/small values |

Floating-point is more flexible but arithmetic hardware is more complex.

Fixed-point is faster/smaller but the programmer must manage scale and overflow carefully.

---

# 14. Exam-Focused Summary

Remember these points:

- Digital building blocks demonstrate hierarchy, modularity, and regularity.
- Ripple-carry adder is simple but slow.
- Carry-lookahead uses generate and propagate signals.
- Prefix adders compute carries using parallel tree logic.
- ALU performs arithmetic and logical operations.
- ALU flags: `N`, `Z`, `C`, `V`.
- `beq` relies on subtraction and the zero flag.
- Shifts can multiply/divide by powers of 2.
- Signed and unsigned operations interpret the same bits differently.

---

# 15. Likely MCQs from Chapter 5

1. Which adder is slow because carry ripples through all stages?
   - Ripple-carry adder.

2. What is `Gi` in carry-lookahead addition?
   - `Gi = AiBi`, meaning the bit position generates a carry.

3. What is `Pi`?
   - `Pi = Ai + Bi`, meaning the bit position propagates a carry.

4. Which ALU flag is used to check equality after subtraction?
   - Zero flag.

5. What is arithmetic right shift used for?
   - Right shifting signed numbers while preserving the sign bit.

6. What is the difference between `div` and `divu`?
   - `div` is signed division; `divu` is unsigned division.


---

<!-- Source file: Chapter_06_RISCV_Architecture.md -->

# Chapter 6 — RISC-V Architecture, Assembly, Machine Language, and Addressing

## 1. Main Objective

This chapter explains the programmer-visible part of the computer.

The main idea is:

> Architecture is what the programmer sees: instructions, registers, memory, and operand locations.

Microarchitecture is how that architecture is implemented in hardware. Chapter 7 focuses on microarchitecture.

---

# 2. Architecture vs Microarchitecture

| Term | Meaning |
|---|---|
| Architecture | programmer's view of the computer |
| Microarchitecture | hardware implementation of that architecture |

Example:

```text
Architecture: RISC-V instructions such as add, lw, sw, beq
Microarchitecture: datapath, control unit, ALU, register file, memories
```

---

# 3. Assembly Language vs Machine Language

| Language | Meaning |
|---|---|
| Assembly language | human-readable instruction format |
| Machine language | binary format understood by hardware |

Example:

```text
Assembly: add x3, x1, x2
Machine:  32-bit binary instruction
```

Assembly is easier for humans; machine language is what the processor actually executes.

---

# 4. RISC-V Design Principles

The slides emphasize these design principles:

| Principle | Meaning |
|---|---|
| Simplicity favors regularity | keep formats consistent |
| Make the common case fast | optimize frequent operations |
| Smaller is faster | smaller hardware is usually faster |
| Good design demands good compromises | balance flexibility and simplicity |

RISC-V keeps instructions simple so hardware can decode and execute them efficiently.

---

# 5. Basic RISC-V Instruction Format

For arithmetic instructions:

```text
add rd, rs1, rs2
sub rd, rs1, rs2
```

Meaning:

| Field | Meaning |
|---|---|
| `rd` | destination register |
| `rs1` | source register 1 |
| `rs2` | source register 2 |

Example:

```text
add x5, x6, x7   # x5 = x6 + x7
sub x5, x6, x7   # x5 = x6 - x7
```

---

# 6. Registers

RISC-V has 32 general-purpose registers.

Important ABI names:

| Name | Register | Use |
|---|---|---|
| zero | x0 | constant 0 |
| ra | x1 | return address |
| sp | x2 | stack pointer |
| gp | x3 | global pointer |
| tp | x4 | thread pointer |
| t0-t6 | x5-x7, x28-x31 | temporaries |
| s0-s11 | x8-x9, x18-x27 | saved registers |
| a0-a7 | x10-x17 | arguments / return values |

## Common exam trap

`x0` is always 0. Writing to `x0` has no effect.

---

# 7. Memory Operands

Registers are fast but limited. Memory is large but slower.

RISC-V uses load/store architecture:

- arithmetic uses registers
- memory is accessed only by load/store instructions

## Load word

```text
lw rd, offset(rs1)
```

Meaning:

```text
rd = Memory[rs1 + offset]
```

## Store word

```text
sw rs2, offset(rs1)
```

Meaning:

```text
Memory[rs1 + offset] = rs2
```

## Byte addressing

RISC-V is byte-addressable.

A 32-bit word is 4 bytes, so word addresses differ by 4 bytes.

Example:

```text
word 0 address = 0
word 1 address = 4
word 2 address = 8
```

---

# 8. Constants and Immediates

An immediate is a constant stored directly in the instruction.

```text
addi rd, rs1, imm
```

Example:

```text
addi x5, x5, 4    # x5 = x5 + 4
addi x6, x5, -12  # x6 = x5 - 12
```

There is no separate `subi` instruction needed because adding a negative immediate does the same job.

---

# 9. Logical and Shift Instructions

## Logical instructions

| Instruction | Use |
|---|---|
| `and` | masking bits |
| `or` | combining bit fields |
| `xor` | toggling/inverting bits |

Example:

```text
and x5, x6, x7
or  x5, x6, x7
xor x5, x6, x7
```

## Shift instructions

| Instruction | Meaning |
|---|---|
| `sll` | shift left logical |
| `srl` | shift right logical |
| `sra` | shift right arithmetic |
| `slli` | shift left logical immediate |
| `srli` | shift right logical immediate |
| `srai` | shift right arithmetic immediate |

---

# 10. Multiplication and Division Instructions

## Multiplication

```text
mul  rd, rs1, rs2   # lower 32 bits
mulh rd, rs1, rs2   # upper 32 bits, signed
```

## Division and remainder

```text
div  rd, rs1, rs2   # signed quotient
rem  rd, rs1, rs2   # signed remainder
divu rd, rs1, rs2   # unsigned quotient
remu rd, rs1, rs2   # unsigned remainder
```

Your assignment pattern shows that these may be asked by tracing register values in both decimal and hexadecimal.

---

# 11. Branches and Jumps

Branches execute instructions out of sequence.

## Conditional branches

| Instruction | Meaning |
|---|---|
| `beq` | branch if equal |
| `bne` | branch if not equal |
| `blt` | branch if less than |
| `bge` | branch if greater than or equal |

Example:

```text
beq x1, x2, target
```

If `x1 == x2`, execution jumps to `target`.

## Unconditional jumps

| Instruction | Meaning |
|---|---|
| `j label` | jump to label |
| `jal rd, label` | jump and save return address |
| `jalr rd, imm(rs1)` | jump to address in register + immediate |

---

# 12. If Statements and Loops

## If statement pattern

C code:

```c
if (i == j)
    f = g + h;
f = f - i;
```

Assembly idea:

```text
bne i, j, L1      # test opposite condition
add f, g, h
L1:
sub f, f, i
```

Exam tip:

> Assembly often branches on the opposite condition to skip over the if-body.

## While loop pattern

C code:

```c
while (condition) {
    body;
}
```

Assembly idea:

```text
loop:
    branch_if_false done
    body
    j loop
done:
```

## For loop pattern

C code:

```c
for (init; condition; update) {
    body;
}
```

Assembly idea:

```text
init
loop:
    branch_if_false done
    body
    update
    j loop
done:
```

---

# 13. Arrays

Array access uses base address plus index offset.

For an integer array where each element is 4 bytes:

```text
address of A[i] = base address + 4i
```

Assembly pattern:

```text
slli t0, i, 2      # t0 = i * 4
add  t0, base, t0  # address of A[i]
lw   x5, 0(t0)     # x5 = A[i]
```

---

# 14. Function Calls

RISC-V calling convention:

| Purpose | Registers |
|---|---|
| arguments | a0-a7 |
| return value | a0 |
| return address | ra |
| stack pointer | sp |
| temporaries | t0-t6 |
| saved registers | s0-s11 |

## Call and return

```text
jal function_name   # call function, stores return address in ra
jr ra               # return to caller
```

## Stack idea

The stack is used to save registers temporarily.

- stack grows downward
- `sp` points to top of stack
- function creates stack space by subtracting from `sp`
- function releases stack space by adding back to `sp`

Example:

```text
addi sp, sp, -8
sw   ra, 4(sp)
sw   s0, 0(sp)
...
lw   s0, 0(sp)
lw   ra, 4(sp)
addi sp, sp, 8
jr   ra
```

---

# 15. Machine Language Instruction Formats

RISC-V instructions are 32 bits.

Common formats:

| Format | Used by | Important fields |
|---|---|---|
| R-type | `add`, `sub`, `and`, `or`, `slt` | `funct7`, `rs2`, `rs1`, `funct3`, `rd`, `op` |
| I-type | `addi`, `lw`, `jalr` | `imm`, `rs1`, `funct3`, `rd`, `op` |
| S-type | `sw` | immediate split across two fields |
| B-type | `beq`, `bne` | branch immediate split across fields |
| U-type | `lui` | upper 20-bit immediate |
| J-type | `jal` | jump immediate split across fields |

---

# 16. Important Opcodes and funct Fields

| Instruction | Opcode decimal | Opcode binary | funct3 | funct7 | Type |
|---|---:|---|---|---|---|
| add | 51 | 0110011 | 000 | 0000000 | R |
| sub | 51 | 0110011 | 000 | 0100000 | R |
| and | 51 | 0110011 | 111 | 0000000 | R |
| or | 51 | 0110011 | 110 | 0000000 | R |
| addi | 19 | 0010011 | 000 | - | I |
| beq | 99 | 1100011 | 000 | - | B |
| bne | 99 | 1100011 | 001 | - | B |
| lw | 3 | 0000011 | 010 | - | I |
| sw | 35 | 0100011 | 010 | - | S |
| jal | 111 | 1101111 | - | - | J |
| jalr | 103 | 1100111 | 000 | - | I |
| lui | 55 | 0110111 | - | - | U |

---

# 17. Addressing Modes

Addressing mode means how operands are located.

| Addressing mode | Used by | Example |
|---|---|---|
| Register only | R-type | `add x3, x1, x2` |
| Immediate | I-type ALU | `addi x3, x1, -5` |
| Base addressing | load/store | `lw x5, 8(x3)` |
| PC-relative | branch/jump | `beq x1, x2, label` |

---

# 18. Machine-Code Decoding Method

When decoding a 32-bit machine instruction:

1. Convert hex to binary.
2. Extract opcode bits `[6:0]`.
3. Use opcode to identify format.
4. Extract fields based on format.
5. Use `funct3` and `funct7` to identify exact instruction.
6. Convert register numbers to names if needed.
7. Sign-extend immediate if it is signed.

---

# 19. Performance Basics from Chapter 6

Performance questions usually use:

```text
Execution Time = Instruction Count x CPI x Clock Cycle Time
Execution Time = Instruction Count x CPI / Clock Rate
```

## Average CPI

```text
Average CPI = Total cycles / Total instructions
```

or:

```text
Average CPI = sum(frequency_i x CPI_i)
```

## MIPS

```text
MIPS = Clock Rate / (CPI x 10^6)
```

Important warning:

> MIPS can be misleading when comparing different instruction sets or different programs.

---

# 20. Exam-Focused Summary

Remember these points:

- RISC-V is a load/store architecture.
- Arithmetic instructions use registers, not memory operands directly.
- `lw` reads memory into a register.
- `sw` writes a register value into memory.
- Branches often test the opposite condition to skip code.
- `jal` stores `PC+4` in the destination register and jumps.
- `ra` stores the return address for function calls.
- `a0-a7` pass arguments; `a0` also returns values.
- Machine-code decoding starts with opcode.
- Addressing modes: register, immediate, base, PC-relative.

---

# 21. Likely Chapter 6 Question Styles

## MCQ style

- Which addressing mode is used by `lw x5, 8(x3)`?
- Which register always contains zero?
- Which fields distinguish `add` from `sub`?
- Which instruction stores `PC+4` in `rd`?
- Which register holds the function return address?

## Short descriptive style

- Explain architecture vs microarchitecture.
- Explain byte-addressable memory.
- Explain how a `for` loop is translated to RISC-V.
- Explain the stack operations during a function call.

## Numerical/code-trace style

- Trace register values after each instruction.
- Convert a C loop to RISC-V assembly.
- Decode a machine instruction from hex.
- Compute CPI/execution time from instruction count and clock rate.


---

<!-- Source file: Chapter_07_Single_Cycle_Processor.md -->

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


---

<!-- Source file: Chapter_07_Pipelined_Processor.md -->

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


---

<!-- Source file: Exam_Question_Patterns.md -->

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


---

<!-- Source file: Quick_Revision_Cheat_Sheet.md -->

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


---

<!-- Source file: Source_Map.md -->

# Source Map

This note pack was made from the uploaded material.

| Uploaded file | Used for |
|---|---|
| `DIP-Notes-master(1)(3).zip` | formatting style: roadmap, simple language, exam-focused summaries |
| `DDCArv_Ch5.pptx` | Chapter 5 arithmetic circuits, digital building blocks, adders, ALU, flags, shifters, multiplication/division |
| `DDCArv_Ch6.pdf` | Chapter 6 RISC-V architecture, assembly language, machine language, addressing modes, functions, stack, performance basics |
| `DDCArv_Ch7.pptx` | Chapter 7 microarchitecture, single-cycle processor, pipelined processor, hazards, performance |
| `Assignment 1- Spr 2026.pdf` | performance numerical question style |
| `Computer Architecture-A2-v1(1).pdf` | RISC-V tracing, multiply/divide, loops, memory, stack question style |
| `CA Assignment-3 Part1.pdf` | single-cycle datapath/control-signal/dataflow question style |
| `RISCV_Datapath_Question-SectionE.docx` | control-signal table program format |
| `RISCV_Datapath_Question_SectionF.docx` | control-signal table program format |
| `Practice Exercise Pipelined RISC-V.docx` | pipeline timing, forwarding, stalls, flushes, register read/write cycle questions |
| `Comp-Arch-Session-I(solution).pdf` | performance and RISC-V code-tracing sessional style |
| `sessional 2 solution.pdf` | scanned final/session-style evidence for RISC-V ISA and single-cycle/pipelined CLO focus |

These notes are paraphrased and reorganized for revision, not copied slide-by-slide.
