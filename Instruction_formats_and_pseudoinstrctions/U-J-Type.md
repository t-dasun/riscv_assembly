# RISC-V: U-Type & J-Type Instructions 💻

U-type and J-type instructions are designed for handling large immediate values and controlling the program's flow. While they serve different purposes, they share a similar instruction format that includes a destination register (`rd`) and a 20-bit immediate value. This structural similarity helps simplify the processor's decoding hardware.

---

## U-Type (Upper Immediate) Instructions

The primary purpose of U-type instructions is to construct large, 32-bit constants or addresses. They work by loading a 20-bit immediate value into the **upper 20 bits** (bits 31-12) of a destination register, with the lower 12 bits being set to zero.

### **LUI** - Load Upper Immediate
* **Syntax**: `LUI rd, imm`
* **Operation**: `rd = imm << 12`
* **Description**: This is the most straightforward U-type instruction. It takes the 20-bit `imm`, shifts it left by 12 bits (effectively placing it in the upper part of the register), and writes the result to `rd`. The lower 12 bits of `rd` become zero.
* **Use Case**: `LUI` is essential for loading a 32-bit constant. You typically use `LUI` to set the upper 20 bits and then follow it with an I-type instruction like `ADDI` to set the lower 12 bits.
    ```assembly
    # Load the 32-bit value 0xABCDE123 into register t0
    lui t0, 0xABCDE      # t0 = 0xABCDE000
    addi t0, t0, 0x123   # t0 = 0xABCDE000 + 0x123 = 0xABCDE123
    ```

### **AUIPC** - Add Upper Immediate to PC
* **Syntax**: `AUIPC rd, imm`
* **Operation**: `rd = PC + (imm << 12)`
* **Description**: This instruction adds the 20-bit `imm` (shifted left by 12 bits) to the value of the **program counter (PC)** and stores the result in `rd`.
* **Use Case**: `AUIPC` is used for **PC-relative addressing**. It allows you to generate addresses for data or functions that are far away from the current instruction, without needing absolute addresses. This makes code position-independent and easier for the linker to manage.

---

## J-Type (Jump) Instructions

J-type instructions are used for unconditional jumps, changing the program's execution flow. The 20-bit immediate is sign-extended and added to the PC to calculate the target address.

### **JAL** - Jump and Link
* **Syntax**: `JAL rd, imm`
* **Operation**:
    1.  **Link**: `rd = PC + 4`
    2.  **Jump**: `PC = PC + imm`
* **Description**: `JAL` is the primary instruction for making function calls. It performs two critical actions simultaneously:
    * It saves the address of the *next* instruction (`PC + 4`) into the destination register `rd`. This is the **return address**.
    * It jumps to a new location by adding the sign-extended immediate `imm` to the current PC.
* **Convention**: The default register for `rd` is typically `x1`, which is also known as the **return address register (`ra`)**.

---

## Pseudoinstructions

Pseudoinstructions are convenient shorthands provided by the assembler that translate into real hardware instructions. They make assembly code cleaner and more intuitive. 🧐

### **J** - Jump
* **Syntax**: `J offset`
* **Description**: This provides a simple, unconditional jump to a label or offset without saving a return address.
* **Translation**: The assembler converts this into a `JAL` instruction but uses `x0` (the zero register) as the destination. Since any value written to `x0` is discarded, the "link" step is effectively ignored.
    * `J offset`  ➡️  `JAL x0, offset`

### **JAL** (implicit `rd`)
* **Syntax**: `JAL offset`
* **Description**: This is a more convenient way to write a standard function call. When the destination register is omitted, the assembler assumes you want to use the standard return address register, `x1` (`ra`).
* **Translation**:
    * `JAL offset`  ➡️  `JAL x1, offset`