# RISC-V Privileged ISA: CSR Functions

This note provides an overview of the Control and Status Registers (CSRs) and their functions at the Machine and Supervisor privilege levels in the RISC-V architecture. CSRs are special registers used to control the processor's state and manage features like interrupts, exceptions, and virtual memory.

---

## Machine Level (M-mode)

M-mode is the **highest privilege level** in RISC-V, providing direct and unrestricted access to hardware. It's used for low-level tasks like handling system resets, managing hardware interrupts, and secure execution.

### Key M-mode Instructions

* `MRET`: **Machine Return**. Used to return from an M-mode trap handler (exception or interrupt) to the program that was interrupted. It restores the privilege level and interrupt-enable state.
* `WFI`: **Wait for Interrupt**. Pauses the processor (hart) until an interrupt occurs, saving power.

### Core M-mode CSRs

M-mode code uses several CSRs to manage the system's state:

* **`misa` (Machine ISA Register):** Describes the supported base ISA (e.g., RV32, RV64 via the `MXL` field) and any implemented standard extensions. Its lower (WARL) 26 bits correspond to letters 'A' through 'Z', where a set bit indicates that the corresponding extension is supported.  leftmost WARL 2-bit field called MXL contains 1 if at reset RV32 is running, 2 if RV64 is running, and 3 if RV128 is running. 

| Bit | Character | Description |
|:---:|:---:|:---|
| 0 | A | Atomic extension |
| 1 | B | Tentatively reserved for Bit-Manipulation extension |
| 2 | C | Compressed extension |
| 3 | D | Double-precision floating-point extension |
| 4 | E | RV32E base ISA |
| 5 | F | Single-precision floating-point extension |
| 6 | G | Reserved |
| 7 | H | Hypervisor extension |
| 8 | I | RV32I/64I/128I base ISA |
| 9 | J | Tentatively reserved for Dynamically Translated Languages extension |
| 10 | K | Reserved |
| 11 | L | Reserved |
| 12 | M | Integer Multiply/Divide extension |
| 13 | N | Tentatively reserved for User-Level Interrupts extension |
| 14 | O | Reserved |
| 15 | P | Tentatively reserved for Packed-SIMD extension |
| 16 | Q | Quad-precision floating-point extension |
| 17 | R | Reserved |
| 18 | S | Supervisor mode implemented |
| 19 | T | Reserved |
| 20 | U | User mode implemented |
| 21 | V | Tentatively reserved for Vector extension |
| 22 | W | Reserved |
| 23 | X | Non-standard extensions present |
| 24 | Y | Reserved |
| 25 | Z | Reserved |

* **`mstatus` (Machine Status):** A critical register that holds the current operating state. Key fields include (always 64bits, RV32 machines implement it as two separate CSRs, mstatus and an additional mstatush containing the higher 32 bits):
xIE, xPIE, xPP here x = M (machine mode)
    * `MIE` (Machine Interrupt Enable): A global switch to enable or disable interrupts in M-mode.
    * `MPIE` (Machine Previous Interrupt Enable): Stores the value of `MIE` before a trap occurred.
    * `MPP` (Machine Previous Privilege): Stores the privilege level (e.g., User, Supervisor) the processor was in before entering M-mode.
* **`mie` & `mip` (Machine Interrupt Enable & Pending):** These WARL registers work together to handle interrupts.
    * `mie`: Determines which specific interrupts (e.g., timer, external) are enabled.
    * `mip`: Shows which interrupts are currently pending (waiting to be handled). An interrupt is only taken if its corresponding bit is set in both `mie` and `mip`.
* **`mtvec` (Machine Trap Vector Base Address):** Holds the base address of the M-mode trap handler code. When a trap occurs, the Program Counter (PC) is set to this address.
* **`mepc` (Machine Exception Program Counter):** When a trap occurs, this register saves the address of the instruction that was interrupted, allowing the `MRET` instruction to resume execution at the correct location.
* **`mcause` (Machine Cause):** Records the reason for the trap. It indicates whether the trap was an **interrupt** or a synchronous **exception** (e.g., illegal instruction) and provides a specific code for the cause.
* **`mtval` (Machine Trap Value):** Provides additional information about the trap. For example, in a memory access fault, it would store the faulty memory address.
* **`mconfigptr` (Machine Configuration Pointer):** contains the address to the configuration data structure that can be read by software to retrieve information about the configuration of harts and the platform.

---

## Supervisor Level (S-mode)

S-mode is the privilege level below M-mode, designed primarily for running an **Operating System (OS)**. Its main responsibility is to provide a stable execution environment for user applications, chiefly by managing **virtual memory**.

### Virtual Memory Abstraction

Virtual memory allows an OS to present each application with its own private, contiguous address space (a "virtual" view of memory). The OS and hardware work together to **translate** these virtual addresses into actual physical memory addresses. This is done using data structures called **page tables**, which map 4 KB chunks of virtual memory (pages) to physical memory.

### Key S-mode Instructions

* `SRET`: **Supervisor Return**. The S-mode equivalent of `MRET`. It's used to return from a trap that was handled in S-mode (e.g., a page fault).
* `SFENCE.VMA`: **Supervisor Fence for Virtual Memory**. This is a memory fence instruction that ensures all prior writes to memory-management data structures (like page tables) are visible to subsequent address translations. It is crucial when the OS modifies page table entries.

### Core S-mode CSRs

S-mode has its own set of CSRs that parallel the M-mode registers. The OS uses these to handle traps that occur in user or supervisor mode without needing to enter the more privileged M-mode.

* `sstatus`, `sie`, `sip`, `sepc`, `scause`, `stval`: These are the S-mode equivalents of `mstatus`, `mie`, `mip`, `mepc`, `mcause`, and `mtval`. They function identically but are accessible from S-mode and manage S-level traps.
* **`satp` (Supervisor Address Translation and Protection):** This is the most important CSR for virtual memory. It controls how virtual-to-physical address translation is performed. Its fields include:
    * `MODE`: Specifies the virtual memory scheme being used (e.g., Sv39 for 39-bit virtual addresses on a 64-bit system).
    * `ASID` (Address Space Identifier): Helps distinguish between the address spaces of different processes, preventing the need to flush translation caches on every context switch.
    * `PPN` (Physical Page Number): Contains the physical address of the root page table, which is the starting point for any address translation.