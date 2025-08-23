# RISC-V 'A' Standard Extension: Atomic Memory Operations

The 'A' extension provides instructions for **atomic memory operations**, which are essential for synchronizing multiple hardware threads (harts) that share memory. ⚛️

An atomic operation is a sequence of read, modify, and write actions on a memory location that is guaranteed to execute completely without being interrupted by any other hart.

---

## Core Concepts

### Memory Coherence & Consistency
* **Coherence:** Ensures that all harts will eventually see the same sequence of write operations to a specific memory address, making the system behave like a single, unified memory.
* **Consistency:** Defines the rules for the ordering of memory operations between different harts, ensuring that programs behave predictably when these synchronization rules are followed.

### Acquire and Release Semantics (`aq`/`rl`)
All instructions in the 'A' extension include two optional flags to enforce memory ordering:
* **`aq` (acquire):** Guarantees that no subsequent memory operations are executed before this atomic operation is complete.
* **`rl` (release):** Guarantees that all previous memory operations are completed before this atomic operation begins.

When both `aq` and `rl` are set, the operation is **sequentially consistent**, meaning it cannot be reordered with any other memory operations.

---

## 4 Load-Reserved / Store-Conditional Instructions

This pair of instructions provides a mechanism to build complex atomic sequences, most notably a **Compare-and-Swap (CAS)**.

* **`LR.W rd, rs1`** (Load-Reserved Word): Loads the word from the memory address in `rs1` into `rd` and places a special "reservation" on that memory address.
* **`SC.W rd, rs2, rs1`** (Store-Conditional Word): Attempts to store the value from `rs2` to the memory address in `rs1`. It succeeds only if the reservation from a previous `LR` is still valid. The instruction writes `0` to `rd` on success and a non-zero value on failure.
* **`LR.D` / `SC.D`**: **RV64 only** versions that operate on doublewords (64 bits).

### Example: Implementing Compare-and-Swap
This sequence shows how `LR`/`SC` can be used to atomically swap the value at `M[x2]` with the value in `x4`, but only if the original value at `M[x2]` matches the expected value in `x3`.

| Instruction         | Comment                                                                                                     |
| :------------------ | :---------------------------------------------------------------------------------------------------------- |
| `LR.W x1, (x2)`       | Loads the value from memory address `x2` into `x1` and reserves the address.                                |
| `BNE x1, x3, FAILED`  | If the loaded value (`x1`) does not match the expected value (`x3`), the swap fails and branches to `FAILED`.   |
| `SC.W x1, x4, (x2)`   | Tries to store the new value from `x4` into the memory at `x2`. `x1` becomes `0` on success.                    |
| `BNEZ x1, RETRY`      | If the store failed (`x1` is not zero), it means another hart interfered, so we branch back to `RETRY` the loop. |

---

## 18 Atomic Memory Operations (AMOs)

AMOs provide common read-modify-write patterns in a single instruction. They atomically:
1. Load a value from the memory address in `rs1` into `rd`.
2. Perform an operation between this loaded value and the value in `rs2`.
3. Write the result of the operation back to the memory address in `rs1`.

### Arithmetic & Swap
* **`AMOADD.W`/`.D`**: Performs an atomic add. `M[rs1] ← M[rs1] + rs2`.
* **`AMOSWAP.W`/`.D`**: Atomically swaps the value in memory with a register. `M[rs1] ↔ rs2`.

### Logical Operations
* **`AMOAND.W`/`.D`**: Atomic logical AND.
* **`AMOOR.W`/`.D`**: Atomic logical OR.
* **`AMOXOR.W`/`.D`**: Atomic logical XOR.

### Comparison Operations
* **`AMOMAX.W`/`.D`**: Stores the maximum of the memory value and `rs2` (**signed**).
* **`AMOMAXU.W`/`.D`**: Stores the maximum of the memory value and `rs2` (**unsigned**).
* **`AMOMIN.W`/`.D`**: Stores the minimum of the memory value and `rs2` (**signed**).
* **`AMOMINU.W`/`.D`**: Stores the minimum of the memory value and `rs2` (**unsigned**).