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
