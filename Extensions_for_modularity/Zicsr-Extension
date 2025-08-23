# RISC-V 'Zicsr' and 'Zifencei' Extensions

## 'Zicsr': Control and Status Register Instructions 📜
This extension provides the standard instructions needed to read and write the Control and Status Registers (CSRs), which are used to configure and monitor the processor's state.

---

### Register-Based CSR Instructions
These instructions use a general-purpose register (`rs1`) as a data source.

* **`CSRRW rd, csr, rs1`** (CSR Read and Write): Atomically swaps values. The original value of the `csr` is written to `rd`, and the value from `rs1` is written to the `csr`.
* **`CSRRS rd, csr, rs1`** (CSR Read and Set): Reads the `csr` value into `rd`, then sets bits in the `csr`. The value in `rs1` acts as a bitmask; for each `1` in `rs1`, the corresponding bit in the `csr` is set to `1`.
* **`CSRRC rd, csr, rs1`** (CSR Read and Clear): Reads the `csr` value into `rd`, then clears bits in the `csr`. The value in `rs1` acts as a bitmask; for each `1` in `rs1`, the corresponding bit in the `csr` is cleared to `0`.

### Immediate-Based CSR Instructions
These instructions are similar but use a 5-bit immediate value (zero-extended to `XLEN`) instead of a register.

* **`CSRRWI rd, csr, imm`**: Reads the `csr` into `rd` and writes the immediate value to the `csr`.
* **`CSRRSI rd, csr, imm`**: Reads the `csr` into `rd` and uses the immediate as a bitmask to **set** bits.
* **`CSRRCI rd, csr, imm`**: Reads the `csr` into `rd` and uses the immediate as a bitmask to **clear** bits.

### Pseudo-instructions
These are common operations provided by assemblers for convenience. They are implemented by using the `x0` (zero) register to ignore a read or a write.

* **`CSRR rd, csr`** (CSR Read): Reads the value of `csr` into `rd`. Expands to `CSRRS rd, csr, x0`.
* **`CSRW csr, rs1`** (CSR Write): Writes the value of `rs1` into `csr`. Expands to `CSRRW x0, csr, rs1`.
* **`CSRS csr, rs1`** (CSR Set): Sets bits in `csr` using `rs1` as a mask. Expands to `CSRRS x0, csr, rs1`.
* **`CSRC csr, rs1`** (CSR Clear): Clears bits in `csr` using `rs1` as a mask. Expands to `CSRRC x0, csr, rs1`.
* **`CSRSI csr, imm`** (CSR Set Immediate): Sets bits in `csr` using an immediate. Expands to `CSRRSI x0, csr, imm`.
* **`CSRCI csr, imm`** (CSR Clear Immediate): Clears bits in `csr` using an immediate. Expands to `CSRRCI x0, csr, imm`.

**Note:** Most CSR write operations are **privileged** and can only be executed in higher-privilege modes (like machine mode).

---

## 'Zifencei': Instruction-Fetch Fence 🚧
This small extension adds a single, important instruction for synchronization.

### The `FENCE.I` Instruction
* **Syntax**: `FENCE.I`
* **Purpose**: To synchronize the instruction and data streams.
* **Function**: It acts as a memory barrier that ensures all data stores completed **before** the `FENCE.I` are visible to the instruction fetch unit **after** the `FENCE.I`.
* **Use Case**: This is critical for systems that modify their own code, such as Just-In-Time (JIT) compilers or self-updating programs. It guarantees that the processor fetches the newly written instructions, not the old, stale ones.

This is different from the standard `FENCE` instruction, which synchronizes **data memory** accesses between different harts. `FENCE.I` synchronizes writes to instruction memory with the processor's own instruction fetch logic.