
# RISC-V 'C' Standard Extension: Compressed Instructions

The 'C' extension, or RVC, provides **16-bit** instruction encodings for common 32-bit base instructions. Its primary goal is to **reduce code size**, which is crucial for embedded systems and can improve instruction cache performance. ⚙️

The 'C' extension is not a standalone ISA; it must be used with a base like RV32 or RV64. It achieves its compression by encoding frequently used variations of larger instructions where some operands can be implied.

---

## How Compression is Achieved

The 16-bit instruction format saves space by making assumptions about the operands, such as:
* **Implicit `zero` Register:** Using the `zero` register (`x0`) as an operand without encoding it. For example, `c.li rd, imm` aliases to `addi rd, x0, imm`.
* **Implicit ABI Registers:** Assuming the use of specific registers defined by the Application Binary Interface (ABI), like the stack pointer `sp` (`x2`). For example, `c.lwsp rd, offset` aliases to `lw rd, offset(x2)`.
* **Shared Destination/Source:** Using the same register for both a source operand and the destination. For example, `c.addi rd, imm` aliases to `addi rd, rd, imm`.
* **Smaller Register Set:** Some instructions only operate on a popular subset of registers (`x8` to `x15`). In assembly, these are marked with a prime (e.g., `rd'`).

---

## C Extension Instruction Aliases

The following table shows the 16-bit compressed instructions and the 32-bit base instructions they expand to.

| Type | Instruction Name                            | Compressed Assembly Syntax  | Corresponding RV32 Instruction      |
| :--- | :------------------------------------------ | :-------------------------- | :------------------------------------ |
| CIW  | Compressed Add Imm\*4 to Stack Pointer      | `c.addi4spn rd' imm`        | `addi rd', x2, imm`                  |
| CL   | Compressed Load Word                        | `c.lw rd' offset`           | `lw rd' offset(rs')`                |
| CA   | Compressed And                              | `c.and rd' rs'`             | `and rd' rd' rs'`                   |
|      | Compressed Or                               | `c.or rd' rs'`              | `or rd' rd' rs'`                    |
|      | Compressed Xor                              | `c.xor rd' rs'`             | `xor rd' rd' rs'`                   |
|      | Compressed Subtract                         | `c.sub rd' rs'`             | `sub rd' rd' rs'`                   |
| CB   | Compressed Branch if Equal to Zero          | `c.beqz rs' imm`            | `beq rs' x0 imm`                    |
|      | Compressed Branch if Not Equal to Zero      | `c.bnez rs' imm`            | `bne rs' x0 imm`                    |
|      | Compressed And Immediate                    | `c.andi rd' imm`            | `andi rd' rd' imm`                  |
|      | Compressed Shift Right Logical Immediate    | `c.srli rd' shamt`          | `srli rd' rd' shamt`                |
|      | Compressed Shift Right Arithmetic Immediate | `c.srai rd' shamt`          | `srai rd' rd' shamt`                |
| CJ   | Compressed Jump                             | `c.j offset`                | `jal x0 offset`                     |
|      | Compressed Jump and Link                    | `c.jal offset`              | `jal x1 offset`                     |
|      | Compressed No Operation                     | `c.nop`                     | `nop`                               |
| CI   | Compressed Add Immediate                    | `c.addi rd imm`             | `addi rd rd imm`                    |
|      | Compressed Shift Left Logical Immediate     | `c.slli rd, shamt`          | `slli rd rd shamt`                  |
|      | Compressed Load Immediate                   | `c.li rd imm`               | `addi rd x0 imm`                    |
|      | Compressed Load Upper Immediate             | `c.lui rd, imm`             | `lui rd imm`                        |
| CSS  | Compressed Load Word from SP                | `c.lwsp rd offset`          | `lw rd offset(x2)`                  |
|      | Compressed Store Word to SP                 | `c.swsp rd offset`          | `sw rd offset(x2)`                  |
| CR   | Compressed Add                              | `c.add rd rs`               | `add rd rd rs`                      |
|      | Compressed Move                             | `c.mv rd rs`                | `add rd x0 rs`                      |
|      | Compressed Jump Register                    | `c.jr rs`                   | `jalr x0 0(rs)`                     |
|      | Compressed Jump and Link Register           | `c.jalr rs`                 | `jalr x1 0(rs)`                     |
|      | Compressed Environment Break                | `c.ebreak`                  | `ebreak`                            |