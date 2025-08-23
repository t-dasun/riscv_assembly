# RISC-V 'D' Standard Extension: Double-Precision Floating-Point

The 'D' extension provides support for **double-precision (64-bit)** floating-point operations, building upon the single-precision capabilities of the 'F' extension. As such, the 'F' extension is a required prerequisite. 👨‍🔬

---

## Architectural Changes

### `FLEN` Extension
The primary change introduced by the 'D' extension is the widening of the floating-point register file (`f0`-`f31`).
* **`FLEN` is extended from 32 bits to 64 bits.** This allows each `f` register to hold a full double-precision value.



### NaN-Boxing
With 64-bit registers, a technique called **NaN-boxing** becomes possible. This is an efficient method for holding smaller data types within a 64-bit register.
* **Concept:** A 32-bit single-precision float (or another data type) is placed in the lower bits of the 64-bit register. The upper bits are all set to `1`.
* **Result:** According to the IEEE 754 standard, this specific bit pattern is interpreted as a "NaN" (Not a Number). This allows the system to use the same registers to handle multiple data types, such as single-precision floats and other custom values, without ambiguity.

---

## Instructions

The 'D' extension doesn't introduce a large set of entirely new instructions. Instead, it **reuses the instruction set from the 'F' extension**.

The specific precision of an operation (single, double, etc.) is determined by the **`fmt` field** within the instruction's binary encoding. In assembly language, this is represented by changing the instruction's suffix from `.S` (Single) to `.D` (Double).

### Examples of Instruction Changes:
* **Arithmetic:** `FADD.S` becomes `FADD.D`
* **Loads/Stores:** `FLW` (Float Load Word) becomes `FLD` (Float Load Double)
* **Conversions:** `FCVT.W.S` becomes `FCVT.W.D`

Essentially, for most floating-point operations, you simply use the double-precision variant of the corresponding single-precision instruction.