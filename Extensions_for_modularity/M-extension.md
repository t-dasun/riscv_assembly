# RISC-V 'M' Standard Extension: Integer Multiplication and Division

The 'M' extension adds instructions for integer multiplication and division. These are excluded from the base ISA to simplify hardware for low-end applications that may not require them. All instructions in this extension use the R-type format.

---

## Multiplication Operations

These instructions perform multiplication on two source registers.

### `MUL` - Multiply
* **Syntax:** `MUL rd, rs1, rs2`
* **Description:** Multiplies the **signed** values in `rs1` and `rs2` and stores the **lower** `XLEN` bits of the result in `rd`.
    * `rd ← (rs1 × rs2)[XLEN-1:0]`

### `MULH` - Multiply High (Signed)
* **Syntax:** `MULH rd, rs1, rs2`
* **Description:** Multiplies the **signed** values in `rs1` and `rs2` and stores the **upper** `XLEN` bits of the result in `rd`.
    * `rd ← (rs1 × rs2)[2*XLEN-1:XLEN]`

### `MULHSU` - Multiply High (Signed-Unsigned)
* **Syntax:** `MULHSU rd, rs1, rs2`
* **Description:** Multiplies the **signed** value in `rs1` by the **unsigned** value in `rs2` and stores the **upper** `XLEN` bits of the result in `rd`.
    * `rd ← (rs1_signed × rs2_unsigned)[2*XLEN-1:XLEN]`

### `MULHU` - Multiply High (Unsigned)
* **Syntax:** `MULHU rd, rs1, rs2`
* **Description:** Multiplies the **unsigned** values in `rs1` and `rs2` and stores the **upper** `XLEN` bits of the result in `rd`.
    * `rd ← (rs1_unsigned × rs2_unsigned)[2*XLEN-1:XLEN]`

### `MULW` - Multiply Word
* **Syntax:** `MULW rd, rs1, rs2`
* **Description:** (**RV64 Only**) Multiplies the lower 32-bit **signed** values from `rs1` and `rs2` and stores the **sign-extended** 32-bit result in `rd`.
    * `rd ← sign_extend((rs1[31:0] × rs2[31:0])[31:0])`

---

## Division Operations

These instructions perform division on two source registers.

### `DIV` - Divide (Signed)
* **Syntax:** `DIV rd, rs1, rs2`
* **Description:** Divides the **signed** value in `rs1` (dividend) by the **signed** value in `rs2` (divisor) and stores the integer quotient in `rd`.
    * `rd ← rs1 / rs2`

### `DIVU` - Divide (Unsigned)
* **Syntax:** `DIVU rd, rs1, rs2`
* **Description:** Divides the **unsigned** value in `rs1` by the **unsigned** value in `rs2` and stores the quotient in `rd`.
    * `rd ← rs1 / rs2`

### `DIVW` - Divide Word (Signed)
* **Syntax:** `DIVW rd, rs1, rs2`
* **Description:** (**RV64 Only**) Divides the lower 32-bit **signed** value in `rs1` by the lower 32-bit **signed** value in `rs2` and stores the **sign-extended** 32-bit quotient in `rd`.
    * `rd ← sign_extend(rs1[31:0] / rs2[31:0])`

### `DIVUW` - Divide Word (Unsigned)
* **Syntax:** `DIVUW rd, rs1, rs2`
* **Description:** (**RV64 Only**) Divides the lower 32-bit **unsigned** value in `rs1` by the lower 32-bit **unsigned** value in `rs2` and stores the **sign-extended** 32-bit quotient in `rd`.
    * `rd ← sign_extend(rs1[31:0] / rs2[31:0])`

---

## Remainder Operations

These instructions calculate the remainder of a division operation.

### `REM` - Remainder (Signed)
* **Syntax:** `REM rd, rs1, rs2`
* **Description:** Computes the remainder of dividing the **signed** value in `rs1` by the **signed** value in `rs2` and stores it in `rd`.
    * `rd ← rs1 % rs2`

### `REMU` - Remainder (Unsigned)
* **Syntax:** `REMU rd, rs1, rs2`
* **Description:** Computes the remainder of dividing the **unsigned** value in `rs1` by the **unsigned** value in `rs2` and stores it in `rd`.
    * `rd ← rs1 % rs2`

### `REMW` - Remainder Word (Signed)
* **Syntax:** `REMW rd, rs1, rs2`
* **Description:** (**RV64 Only**) Computes the remainder of dividing the lower 32-bit **signed** value in `rs1` by the lower 32-bit **signed** value in `rs2` and stores the **sign-extended** 32-bit result in `rd`.
    * `rd ← sign_extend(rs1[31:0] % rs2[31:0])`

### `REMUW` - Remainder Word (Unsigned)
* **Syntax:** `REMUW rd, rs1, rs2`
* **Description:** (**RV64 Only**) Computes the remainder of dividing the lower 32-bit **unsigned** value in `rs1` by the lower 32-bit **unsigned** value in `rs2` and stores the **sign-extended** 32-bit result in `rd`.
    * `rd ← sign_extend(rs1[31:0] % rs2[31:0])`

---

## Microarchitectural Fusion

The RISC-V ISA suggests that implementations can optionally **fuse** common instruction pairs to improve performance.
* **Full Multiplication:** A `MULH` / `MULHSU` / `MULHU` followed by a `MUL` can be fused to produce the full `2*XLEN` bit result.
* **Division with Remainder:** A `DIV` / `DIVU` followed by a `REM` / `REMU` can be fused, as the division operation typically calculates the remainder as a byproduct.