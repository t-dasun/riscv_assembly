# RISC-V 'F' Standard Extension: Single-Precision Floating-Point

The 'F' extension provides RISC-V with instructions and registers for **single-precision floating-point** arithmetic, adhering to the IEEE 754-2008 standard. 📐

---

## Floating-Point Architecture

### Register File
* The 'F' extension introduces a **separate register file** for floating-point values, named `f0` through `f31`.
    * Register File: This is a hardware term. It describes the physical structure—an indexed array of registers bundled together. same as int general perpose regs x0-x31.
* This keeps floating-point operations separate from integer operations, which can simplify the CPU hardware design (e.g., by allowing a decoupled Floating-Point Unit (FPU)).
    * A Floating-Point Unit (FPU) is a specialized part of a processor designed specifically to handle calculations with floating-point numbers.
* The width of these registers is called **FLEN**, which is 32 bits for the 'F' extension.

### Control and Status Register (`fcsr`)
* A new Control and Status Register (CSR) call `fcsr`, is added to manage the FPU.
* It holds the current **rounding mode** and any **exception flags** (like overflow, underflow, or divide by zero).
* Accessing this register requires the "Zicsr" extension.

---

## Instructions

The 'F' extension adds instructions for moving data, arithmetic, and comparisons.

### Load and Store
These instructions move 32-bit single-precision float values between memory and the floating-point register file.
* **`FLW rd, rs1, offset`** (Float Load Word): Loads a word from memory into float register `rd`. (I type)
* **`FSW rd, rs1, offset`** (Float Store Word): Stores a word from float register `rd` into memory. (S type)

### Arithmetic Operations
These instructions perform standard floating-point calculations. They all use the R-type format.
* **`FADD rd, rs1, rs2`**: `rd ← rs1 + rs2`
* **`FSUB rd, rs1, rs2`**: `rd ← rs1 - rs2`
* **`FMUL rd, rs1, rs2`**: `rd ← rs1 × rs2`
* **`FDIV rd, rs1, rs2`**: `rd ← rs1 / rs2`
* **`FSQRT rd, rs`**: `rd ← √rs` (Note: `FSQRT` only uses one source register).

### Comparison Operations
These instructions compare two floating-point values. (R-type format)
* **`FMIN rd, rs1, rs2`**: Stores the smaller of the two source values into `rd`.
* **`FMAX rd, rs1, rs2`**: Stores the larger of the two source values into `rd`.

---

## Instruction Fields

### `fmt` (Format) Field
This 2-bit field specifies the precision of the floating-point data.
| `fmt` | Mnemonic | Meaning                   |
| :---- | :------- | :------------------------ |
| `00`  | S        | 32-bit single-precision   |
| `01`  | D        | 64-bit double-precision   |
| `10`  | H        | 16-bit half-precision     |
| `11`  | Q        | 128-bit quad-precision    |

### `rm` (Rounding Mode) Field
This 3-bit field in an instruction's encoding specifies how to round the result of an operation if it isn't exact.
| `rm`  | Mnemonic | Meaning                               |
| :---- | :------- | :------------------------------------ |
| `000` | RNE      | Round to Nearest, ties to Even        |
| `001` | RTZ      | Round towards Zero                    |
| `010` | RDN      | Round Down (towards -∞)               |
| `011` | RUP      | Round Up (towards +∞)                 |
| `100` | RMM      | Round to Nearest, ties to Max Magnitude |
| `111` | DYN      | Use the dynamic mode from the `fcsr`  |


----------------------------------------------------------------
----------------------------------------------------------------

# RISC-V 'F' Extension: Additional Instructions

This document covers the advanced arithmetic, conversion, and utility instructions in the single-precision floating-point ('F') extension.

---

## Fused Multiply-Add Instructions

These instructions perform a multiplication and an addition/subtraction in a single, fused operation. This is faster and more precise than two separate instructions because the intermediate product is not rounded. They use a special R4-type format to accommodate three source registers.

* **`FMADD.S rd, rs1, rs2, rs3`**: `rd ← (rs1 × rs2) + rs3` (float multiply-add)
* **`FMSUB.S rd, rs1, rs2, rs3`**: `rd ← (rs1 × rs2) - rs3`
* **`FNMADD.S rd, rs1, rs2, rs3`**: `rd ← -(rs1 × rs2) + rs3` (Negated Multiply-Add)
* **`FNMSUB.S rd, rs1, rs2, rs3`**: `rd ← -(rs1 × rs2) - rs3` (Negated Multiply-Subtract)

---

## Sign-Injection Instructions

These instructions create a new floating-point number in `rd` by combining the magnitude of `rs1` with a sign bit derived from `rs1` and `rs2`. (R-type)

* **`FSGNJ.S rd, rs1, rs2`**: "Sign-Inject" - Sets `rd` to the value of `rs1` with the sign of `rs2`.
* **`FSGNJN.S rd, rs1, rs2`**: "Sign-Inject-Negate" - Sets `rd` to the value of `rs1` with the *inverted* sign of `rs2`.
* **`FSGNJX.S rd, rs1, rs2`**: "Sign-Inject-XOR" - Sets `rd` to the value of `rs1` with a sign that is the XOR of the signs of `rs1` and `rs2`.

---

## Conversion Instructions

These instructions convert values between floating-point and integer formats.(R-type)

### Float-to-Integer (`FCVT.int.S`)
* **`FCVT.W.S rd, rs1`**: Converts float in `rs1` to a 32-bit **signed** integer in `rd`.
* **`FCVT.WU.S rd, rs1`**: Converts float in `rs1` to a 32-bit **unsigned** integer in `rd`.
* **`FCVT.L.S rd, rs1`**: (**RV64 Only**) Converts float in `rs1` to a 64-bit **signed** integer in `rd`.
* **`FCVT.LU.S rd, rs1`**: (**RV64 Only**) Converts float in `rs1` to a 64-bit **unsigned** integer in `rd`.

### Integer-to-Float (`FCVT.S.int`)
* **`FCVT.S.W rd, rs1`**: Converts 32-bit **signed** integer in `rs1` to a float in `rd`.
* **`FCVT.S.WU rd, rs1`**: Converts 32-bit **unsigned** integer in `rs1` to a float in `rd`.
* **`FCVT.S.L rd, rs1`**: (**RV64 Only**) Converts 64-bit **signed** integer in `rs1` to a float in `rd`.
* **`FCVT.S.LU rd, rs1`**: (**RV64 Only**) Converts 64-bit **unsigned** integer in `rs1` to a float in `rd`.

---

## Integer and Floating-Point Move Instructions

These instructions move the raw 32-bit pattern between the integer and floating-point register files **without any conversion or modification**.(R-type)

* **`FMV.X.W rd, rs1`**: Moves the bit pattern from float register `rs1` to integer register `rd`.
* **`FMV.W.X rd, rs1`**: Moves the bit pattern from integer register `rs1` to float register `rd`.

---

## Comparison and Classification

### Comparison Instructions
These instructions compare two float registers (`rs1`, `rs2`) and write a boolean result (`1` for true, `0` for false) to an **integer** register `rd`.(R-type)

* **`FEQ.S rd, rs1, rs2`**: `rd ← (rs1 == rs2)`
* **`FLT.S rd, rs1, rs2`**: `rd ← (rs1 < rs2)`
* **`FLE.S rd, rs1, rs2`**: `rd ← (rs1 <= rs2)`

### Classification Instruction
* **`FCLASS.S rd, rs1`**: "Float Classify" - Examines the value in float register `rs1` and writes a 10-bit mask to **integer** register `rd` to classify its type (e.g., zero, infinity, NaN). Each bit corresponds to a specific property.(R-type)

| `rd` bit | Meaning of Property                  |
| :------- | :----------------------------------- |
| `0`      | `rs1` is negative infinity (-∞)      |
| `1`      | `rs1` is a negative normal number    |
| `2`      | `rs1` is a negative subnormal number |
| `3`      | `rs1` is negative zero (-0)          |
| `4`      | `rs1` is positive zero (+0)          |
| `5`      | `rs1` is a positive subnormal number |
| `6`      | `rs1` is a positive normal number    |
| `7`      | `rs1` is positive infinity (+∞)      |
| `8`      | `rs1` is a signaling NaN (sNaN)      |
| `9`      | `rs1` is a quiet NaN (qNaN)          |

---

## Floating-Point Pseudo-instructions

These are common operations that assemblers provide for convenience. They are implemented using the real sign-injection instructions.

* **`FMV.S rd, rs`** (Float Move): Implemented as `FSGNJ.S rd, rs, rs`.
* **`FNEG.S rd, rs`** (Float Negate): Implemented as `FSGNJN.S rd, rs, rs`.
* **`FABS.S rd, rs`** (Float Absolute Value): Implemented as `FSGNJX.S rd, rs, rs`.