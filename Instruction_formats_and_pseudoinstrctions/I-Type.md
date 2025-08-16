# RISC-V: I-Type Instructions 📝

I-type ("Immediate") instructions are a versatile and frequently used category in the RISC-V ISA. They perform operations using one source register (`rs1`) and an **immediate** (a constant value encoded directly into the instruction), storing the result in a destination register (`rd`).

There are 23 base instructions that use the I-type format, which can be grouped into several functional categories.

---

## Register-Immediate Operations

These 13 instructions follow the syntax `INSTR rd, rs1, imm` and perform an operation between a register and an immediate value.

### **Arithmetic (2)**
* **ADDI / ADDIW**: Add Immediate
    * **Operation**: `rd = rs1 + imm`
    * **Description**: Adds a sign-extended 12-bit immediate to the value in `rs1`. This is also commonly used for moving a value (`MV`), loading an immediate (`LI`), and for no-ops (`NOP`). The `ADDIW` version is for 64-bit systems, operating on 32-bit values and sign-extending the result.

### **Logical (3)**
* **ANDI**: AND Immediate
    * **Operation**: `rd = rs1 & imm`
    * **Description**: Performs a bitwise AND between `rs1` and a sign-extended immediate.

* **ORI**: OR Immediate
    * **Operation**: `rd = rs1 | imm`
    * **Description**: Performs a bitwise OR between `rs1` and a sign-extended immediate.

* **XORI**: XOR Immediate
    * **Operation**: `rd = rs1 ^ imm`
    * **Description**: Performs a bitwise exclusive OR between `rs1` and a sign-extended immediate.

### **Shift by Immediate (6)**
* **SLLI / SLLIW**: Shift Left Logical Immediate
    * **Operation**: `rd = rs1 << imm`
    * **Description**: Shifts `rs1` left by the immediate amount.

* **SRLI / SRLIW**: Shift Right Logical Immediate
    * **Operation**: `rd = rs1 >> imm`
    * **Description**: Shifts `rs1` right, filling the new bits with zeros (for unsigned numbers).

* **SRAI / SRAIW**: Shift Right Arithmetic Immediate
    * **Operation**: `rd = rs1 >> imm`
    * **Description**: Shifts `rs1` right, filling the new bits with the original sign bit (for signed numbers).

### **Comparison (2)**
* **SLTI**: Set Less Than Immediate (Signed)
    * **Operation**: `rd = (rs1 < imm) ? 1 : 0`
    * **Description**: Compares `rs1` and the sign-extended `imm` as **signed** numbers.

* **SLTIU**: Set Less Than Immediate Unsigned
    * **Operation**: `rd = (rs1 < imm) ? 1 : 0`
    * **Description**: Compares `rs1` and the sign-extended `imm` as **unsigned** numbers. `SEQZ` is a common pseudoinstruction based on this.

---

## Load Instructions

These 7 instructions load data from memory into a register. They use the syntax `LOAD rd, offset(rs1)`, where the memory address is calculated as `rs1 + offset`.

* **LB / LBU**: Load Byte (8 bits)
    * Loads a single byte. **`LB` sign-extends** it to the register width, while **`LBU` zero-extends** it.

* **LH / LHU**: Load Halfword (16 bits)
    * Loads two bytes. **`LH` sign-extends** the value, while **`LHU` zero-extends** it.

* **LW / LWU**: Load Word (32 bits)
    * Loads four bytes. On 64-bit systems, **`LW` sign-extends** the 32-bit word, while **`LWU` zero-extends** it.

* **LD**: Load Doubleword (64 bits)
    * Loads eight bytes into a register on a 64-bit system.

---

## Control Transfer & System Instructions

### **JALR** - Jump and Link Register
* **Syntax**: `JALR rd, rs1, imm`
* **Description**: This is an indirect jump instruction with two key actions:
    1.  **Link**: It saves the address of the next instruction (`PC + 4`) into `rd`.
    2.  **Jump**: It sets the PC to the address calculated by `rs1 + imm`, ensuring the last bit is 0.
* **Use Case**: Essential for returning from functions (`ret`) and implementing jump tables.

### **FENCE** - Memory Ordering
* **Syntax**: `FENCE pred, succ`
* **Description**: Guarantees that memory operations (reads/writes) listed in the `pred` set that appear before the `FENCE` are completed before the memory operations in the `succ` set that appear after the `FENCE`. This is crucial for synchronization in multi-core systems.

### **Environment Operations**
* **ECALL**: Environment Call
    * **Description**: Used to make a system call, transferring control to the operating system or execution environment to request a privileged service.

* **EBREAK**: Environment Breakpoint
    * **Description**: Used by debuggers to insert a breakpoint, transferring control to a debugging environment.

---

## Common I-Type Pseudoinstructions

These are assembler shorthands that make code more readable. 💡

* **NOP** (No Operation)
    * Does nothing.
    * **Translates to**: `ADDI x0, x0, 0`

* **LI rd, imm** (Load Immediate)
    * Loads an immediate value into a register.
    * **Translates to**: `ADDI rd, x0, imm` (for small immediates)

* **MV rd, rs** (Move)
    * Moves the content of `rs` to `rd`.
    * **Translates to**: `ADDI rd, rs, 0`

* **SEXT.W rd, rs** (Sign Extend Word)
    * Sign-extends a 32-bit value in `rs` to 64 bits in `rd`.
    * **Translates to**: `ADDIW rd, rs, 0`

* **SEQZ rd, rs** (Set if Equal to Zero)
    * Sets `rd` to 1 if `rs` is 0, otherwise sets `rd` to 0.
    * **Translates to**: `SLTIU rd, rs, 1`

* **FENCE** (no arguments)
    * Fences all memory and I/O.
    * **Translates to**: `FENCE iorw, iorw`

* **JR rs** (Jump Register)
    * Jumps to the address in `rs` without saving a return address.
    * **Translates to**: `JALR x0, rs, 0`

* **JALR rs** (Jump and Link Register, implicit `rd`)
    * Jumps to `rs` and saves the return address in `x1` (`ra`).
    * **Translates to**: `JALR x1, rs, 0`

* **RET** (Return)
    * Returns from a subroutine. Jumps to the address stored in the return address register (`x1`).
    * **Translates to**: `JALR x0, x1, 0`