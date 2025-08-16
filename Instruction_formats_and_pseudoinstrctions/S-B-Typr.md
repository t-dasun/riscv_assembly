# RISC-V: S-Type & B-Type Instructions 💾

S-type ("Store") and B-type ("Branch") instructions are fundamental for memory operations and program control flow. They share a similar format, using two source registers (`rs1` and `rs2`) and a 12-bit immediate, but no destination register.

They are classified as separate types because of how they encode their immediate values. This design choice, while seemingly complex, cleverly simplifies the processor's decoding hardware by maximizing the overlap of bit positions between the different instruction types.

* **S-type immediates** are simple 12-bit signed offsets for memory addressing.
* **B-type immediates** represent a 13-bit signed byte offset (shifted left by one bit), as branch targets are always aligned to a two-byte boundary.

---

## S-Type (Store) Instructions

S-type instructions are used to **store** data from a register into memory.

* **Assembly Syntax**: `STORE rs2, offset(rs1)`
* **Operation**: The value from the source register `rs2` is written to a memory address. This address is calculated by adding the sign-extended `offset` (the immediate) to the base address stored in `rs1`.
    * **Effective Address** = `rs1 + offset`

There are four store instructions, each for a different data size:

* **SB** (Store Byte)
    * Stores the lowest 8 bits (one byte) from `rs2` into memory.

* **SH** (Store Halfword)
    * Stores the lowest 16 bits (two bytes) from `rs2` into memory.

* **SW** (Store Word)
    * Stores the lowest 32 bits (four bytes) from `rs2` into memory.

* **SD** (Store Doubleword)
    * Stores the full 64 bits from `rs2` into memory (used in 64-bit systems).



---

## B-Type (Branch) Instructions

B-type instructions are **conditional branches**. They compare the values in two registers and, if the condition is true, change the program's flow by jumping to a new address.

* **Assembly Syntax**: `BRANCH rs1, rs2, offset`
* **Operation**: If the comparison between `rs1` and `rs2` is true, the program counter (PC) is updated to `PC + offset`. If the condition is false, execution simply continues to the next instruction in sequence.

There are six base branch instructions:

* **BEQ** (Branch if Equal)
    * Jumps if `rs1 == rs2`.

* **BNE** (Branch if Not Equal)
    * Jumps if `rs1 != rs2`.

* **BLT** (Branch if Less Than)
    * Jumps if `rs1 < rs2`, treating them as **signed** numbers.

* **BLTU** (Branch if Less Than, Unsigned)
    * Jumps if `rs1 < rs2`, treating them as **unsigned** numbers.

* **BGE** (Branch if Greater or Equal)
    * Jumps if `rs1 >= rs2`, treating them as **signed** numbers.

* **BGEU** (Branch if Greater or Equal, Unsigned)
    * Jumps if `rs1 >= rs2`, treating them as **unsigned** numbers.

---

## Branch Pseudoinstructions

To make assembly programming more intuitive, the assembler provides several pseudoinstructions that translate into the base branch instructions. 🧠

### Comparison with Zero (`BRANCH rs, offset`)
These instructions compare a single register against zero by using the hardwired `x0` (zero) register as the second operand.

* **BEQZ** (Branch if Equal to Zero) ➡️ `BEQ rs, x0, offset`
* **BNEZ** (Branch if Not Equal to Zero) ➡️ `BNE rs, x0, offset`
* **BLTZ** (Branch if Less Than Zero) ➡️ `BLT rs, x0, offset`
* **BGEZ** (Branch if Greater or Equal to Zero) ➡️ `BGE rs, x0, offset`
* **BLEZ** (Branch if Less or Equal to Zero) ➡️ `BGE x0, rs, offset`
* **BGTZ** (Branch if Greater Than Zero) ➡️ `BLT x0, rs, offset`

### Inverted Conditions (`BRANCH rs1, rs2, offset`)
These instructions provide logical opposites to the base set by swapping the source registers in the underlying instruction.

* **BGT** (Branch if Greater Than) ➡️ `BLT rs2, rs1, offset`
* **BLE** (Branch if Less or Equal) ➡️ `BGE rs2, rs1, offset`
* **BGTU** (Branch if Greater Than, Unsigned) ➡️ `BLTU rs2, rs1, offset`
* **BLEU** (Branch if Less or Equal, Unsigned) ➡️ `BGEU rs2, rs1, offset`