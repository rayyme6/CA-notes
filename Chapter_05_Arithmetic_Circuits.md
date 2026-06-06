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
