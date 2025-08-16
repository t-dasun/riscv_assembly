# RISC-V: Multi-Operation Pseudoinstructions ⚙️

While most pseudoinstructions are simple aliases for a single hardware instruction, some of the most powerful ones expand into a sequence of two or more instructions. These are essential for handling tasks beyond the scope of a single instruction, such as reaching any 32-bit address in memory for jumps or data access.

---

## Far Jumps and Calls

Standard jump and branch instructions have limited range. To jump to any arbitrary 32-bit address, a two-instruction sequence is needed.

### **CALL** offset
* **Purpose**: To perform a function call to a "far-away" subroutine that a single `JAL` can't reach.
* **Mechanism**: It generates a PC-relative address and saves the return address (`ra`, or `x1`).
    1.  `AUIPC x1, offset_upper`: An `AUIPC` instruction calculates the upper 20 bits of the target address relative to the current PC and stores it in the return address register (`x1`). It also correctly adjusts the value to account for the sign-extension of the next instruction.
    2.  `JALR x1, offset_lower(x1)`: A `JALR` instruction adds the lower 12 bits to the address computed by `AUIPC` to find the final target. It then "links" by saving the address of the next instruction back into `x1`, completing the function call.

### **TAIL** offset
* **Purpose**: To perform a "tail call" to a far-away subroutine. This is an optimization where the current function's stack frame is reused, as the call is the very last thing the function does.
* **Mechanism**: It's similar to `CALL` but crucially **does not** save a new return address.
    1.  `AUIPC x6, offset_upper`: `AUIPC` calculates the upper 20 bits of the address, but uses a temporary register (e.g., `x6`, or `t1`).
    2.  `JALR x0, offset_lower(x6)`: The `JALR` jumps to the final target address, but its destination register is `x0` (the zero register). Since writing to `x0` has no effect, the return address is discarded, effectively replacing the current function's return with the new one's.

---

## Handling Large Addresses and Offsets

Load, store, and immediate instructions are limited by their 12-bit immediate fields. The following pseudoinstructions overcome this limitation by using an `AUIPC` to handle the upper 20 bits of an address and a second instruction to handle the lower 12 bits.



### **LA** - Load Address
* **Syntax**: `LA rd, symbol`
* **Purpose**: Loads the absolute address of a `symbol` (like a global variable or function) into a destination register `rd`. Its behavior changes depending on the type of code being generated.
* **Mechanism**:
    * **For Position-Dependent Code**: It expands to a simple address calculation.
        1. `AUIPC rd, upper_20_bits_of(symbol - PC)`
        2. `ADDI rd, rd, lower_12_bits_of(symbol - PC)`
    * **For Position-Independent Code (PIC)**: It loads the address from the Global Offset Table (GOT), which maps symbols to their absolute addresses at runtime.
        1. `AUIPC rd, upper_20_bits_of(GOT_entry - PC)`
        2. `LD rd, lower_12_bits_of(GOT_entry)(rd)` (or `LW` for 32-bit)

### **LLA** - Load Local Address
* **Syntax**: `LLA rd, symbol`
* **Purpose**: Loads the address of a local symbol. Unlike `LA`, it *always* performs a direct PC-relative calculation, even in position-independent code.
* **Mechanism**: It always expands to the same sequence, calculating the address without accessing memory.
    1. `AUIPC rd, upper_20_bits_of(symbol - PC)`
    2. `ADDI rd, rd, lower_12_bits_of(symbol - PC)`

### Global Loads and Stores
* **Syntax**: `LW rd, symbol` or `SW rs, symbol, rt`
* **Purpose**: To perform a load or store to a global symbol that is too far to be addressed with a single instruction's 12-bit offset.
* **Mechanism**:
    * **Loads (`LD`, `LW`, etc.)**:
        1. `AUIPC rd, upper_20_bits_of(symbol - PC)`: Calculates the upper part of the symbol's address and places it in the destination register.
        2. `LD rd, lower_12_bits_of(symbol)(rd)`: Uses the calculated address as a base, adds the lower 12 bits of the offset, and performs the load.
    * **Stores (`SD`, `SW`, etc.)**: Requires a temporary register (`rt`).
        1. `AUIPC rt, upper_20_bits_of(symbol - PC)`: Calculates the upper part of the symbol's address and places it in a **temporary register**.
        2. `SW rs, lower_12_bits_of(symbol)(rt)`: Uses the address in the temporary register as a base and stores the value from the original source register (`rs`).