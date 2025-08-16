# RISC-V: R-Type Instructions & Pseudoinstructions

R-type (Register-type) instructions are a fundamental part of the RISC-V instruction set architecture (ISA). They perform operations using values from two source registers (**rs1** and **rs2**) and store the result in a single destination register (**rd**).

The general assembly syntax is: `INSTR rd, rs1, rs2`

---

## R-Type Base Instructions

There are 15 primary R-type instructions that perform arithmetic, logical, shift, and comparison operations.

### Arithmetic Operations (4)
These instructions perform standard integer arithmetic. Any resulting overflow is simply ignored, and the lower bits of the result are stored in `rd`.

* **ADD / ADDW**: Add
    * **Syntax**: `ADD rd, rs1, rs2` or `ADDW rd, rs1, rs2`
    * **Operation**: `rd = rs1 + rs2`
    * **Description**: Adds the value in register `rs1` to the value in register `rs2`. The `ADDW` variant is for 64-bit systems and operates on 32-bit values (words), sign-extending the 32-bit result to 64 bits.

* **SUB / SUBW**: Subtract
    * **Syntax**: `SUB rd, rs1, rs2` or `SUBW rd, rs1, rs2`
    * **Operation**: `rd = rs1 - rs2`
    * **Description**: Subtracts the value in register `rs2` from the value in register `rs1`. The `SUBW` variant is the 64-bit word-based version.

### Logic Bitwise Operations (3)
These instructions perform bit-by-bit logical operations.

* **AND**: Bitwise AND
    * **Syntax**: `AND rd, rs1, rs2`
    * **Operation**: `rd = rs1 & rs2`
    * **Description**: Performs a bitwise AND between `rs1` and `rs2`.

* **OR**: Bitwise OR
    * **Syntax**: `OR rd, rs1, rs2`
    * **Operation**: `rd = rs1 | rs2`
    * **Description**: Performs a bitwise OR between `rs1` and `rs2`.

* **XOR**: Bitwise XOR
    * **Syntax**: `XOR rd, rs1, rs2`
    * **Operation**: `rd = rs1 ^ rs2`
    * **Description**: Performs a bitwise exclusive OR between `rs1` and `rs2`.

### Shift Operations (6)
These instructions shift the value in `rs1`. The number of bits to shift is specified by the **lower 5 bits** of register `rs2`.

* **SLL / SLLW**: Shift Left Logical
    * **Syntax**: `SLL rd, rs1, rs2` or `SLLW rd, rs1, rs2`
    * **Operation**: `rd = rs1 << rs2[4:0]`
    * **Description**: Shifts `rs1` to the left. The vacated bits on the right are filled with zeros.

* **SRL / SRLW**: Shift Right Logical
    * **Syntax**: `SRL rd, rs1, rs2` or `SRLW rd, rs1, rs2`
    * **Operation**: `rd = rs1 >> rs2[4:0]`
    * **Description**: Shifts `rs1` to the right. The vacated bits on the left are filled with zeros. This is ideal for unsigned numbers.

* **SRA / SRAW**: Shift Right Arithmetic
    * **Syntax**: `SRA rd, rs1, rs2` or `SRAW rd, rs1, rs2`
    * **Operation**: `rd = rs1 >> rs2[4:0]`
    * **Description**: Shifts `rs1` to the right. The vacated bits on the left are filled with copies of the original most-significant bit (the sign bit). This preserves the sign of a signed number.

### Comparison Operations (2)
These instructions compare two registers and set the destination register to `1` (true) or `0` (false) based on the result.

* **SLT**: Set Less Than (Signed)
    * **Syntax**: `SLT rd, rs1, rs2`
    * **Operation**: `rd = (rs1 < rs2) ? 1 : 0`
    * **Description**: Compares `rs1` and `rs2` as **signed** numbers. If `rs1` is less than `rs2`, `rd` is set to 1; otherwise, it is set to 0.

* **SLTU**: Set Less Than Unsigned
    * **Syntax**: `SLTU rd, rs1, rs2`
    * **Operation**: `rd = (rs1 < rs2) ? 1 : 0`
    * **Description**: Compares `rs1` and `rs2` as **unsigned** numbers. If `rs1` is less than `rs2`, `rd` is set to 1; otherwise, it is set to 0.

---

## R-Type Pseudoinstructions

Pseudoinstructions are not real hardware instructions. They are convenient aliases that the assembler translates into base instructions to make code more readable. They typically use a simpler `INSTR rd, rs` format.

### Arithmetic Pseudoinstructions (2)

* **NEG / NEGW**: Negate
    * **Syntax**: `NEG rd, rs` or `NEGW rd, rs`
    * **Description**: Calculates the two's complement of the value in register `rs`.
    * **Translation**: This is equivalent to subtracting `rs` from zero. The `x0` register in RISC-V is hardwired to zero.
        * `NEG rd, rs` becomes `SUB rd, x0, rs`
        * `NEGW rd, rs` becomes `SUBW rd, x0, rs`

### Comparison Pseudoinstructions (3)

* **SGTZ**: Set if Greater Than Zero
    * **Syntax**: `SGTZ rd, rs`
    * **Description**: Sets `rd` to 1 if `rs` is greater than zero (signed).
    * **Translation**: The condition `rs > 0` is the same as `0 < rs`.
        * `SGTZ rd, rs` becomes `SLT rd, x0, rs`

* **SLTZ**: Set if Less Than Zero
    * **Syntax**: `SLTZ rd, rs`
    * **Description**: Sets `rd` to 1 if `rs` is less than zero (signed).
    * **Translation**: This directly maps to the `SLT` instruction, comparing `rs` with zero.
        * `SLTZ rd, rs` becomes `SLT rd, rs, x0`

* **SNEZ**: Set if Not Equal to Zero
    * **Syntax**: `SNEZ rd, rs`
    * **Description**: Sets `rd` to 1 if `rs` is not equal to zero.
    * **Translation**: An unsigned number is greater than zero if and only if it is not zero. Therefore, this checks if `0 < rs` using an unsigned comparison.
        * `SNEZ rd, rs` becomes `SLTU rd, x0, rs`