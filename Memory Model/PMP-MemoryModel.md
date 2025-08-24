# RISC-V Memory Model & Protection

This note covers the core concepts of the RISC-V memory model, including the rules for memory ordering (RVWMO), the inherent physical properties of memory (PMA), and the configurable access control layer (PMP).

---

## The RISC-V Memory Model (RVWMO) ⛓️

A memory model defines the rules for how memory operations appear to execute, which is especially important in multi-threaded systems. RISC-V uses a **"Weak Memory Ordering"** model called **RVWMO**.

The key takeaway is that while the processor is free to reorder memory operations for performance, the model guarantees correctness through a few simple rules:

* **Single-Thread Order:** Within a single core (hart), memory operations appear to happen in the order they are written in your code.
* **Multi-Thread Synchronization:** Between different cores, this order is **not** guaranteed. Programmers must use explicit instructions like `FENCE` or atomics (from the 'A' extension) to ensure that memory writes from one core are visible to another in a predictable order.
* **Correctness Axioms:** The model is built on axioms that guarantee common-sense behavior, such as ensuring that a read operation gets the value from the most recent write (`Load Value Axiom`) and that all memory operations eventually complete (`Progress Axiom`).

---

## Physical Memory Attributes (PMA) 🗺️

A PMA is an **inherent, fixed characteristic** of a physical region of memory. Think of it as the hardware's fundamental nature, which software cannot change. The classic example is the difference between ROM (read-only) and RAM (read-write); these are different memory regions with different PMAs.

A dedicated hardware unit called the **PMA checker** continuously verifies that memory accesses comply with the attributes of the target region.

Two important PMAs are:

### Alignment

RISC-V memory is byte-addressed. An access is **"aligned"** if its address is a multiple of the size of the data being accessed (e.g., accessing a 4-byte word at an address divisible by 4). A **"misaligned"** access (e.g., accessing a 4-byte word at an address ending in `...01`) requires accessing two separate memory locations. The PMA for a region determines if misaligned accesses are allowed. For example, atomic operations require alignment, and the PMA checker will raise an exception if this is violated.

### Cacheability

PMAs define the caching policy for a memory region, which is managed by M-mode code.
* **Non-cached memory:** Must not be cached.
* **Read-only memory:** Can be cached, but writes are forbidden.
* **Multi-hart memory:** Can be cached but must be kept coherent between cores, using a specified cache coherence protocol to prevent reading stale data.

---

## Physical Memory Protection (PMP) 🛡️

While PMAs describe *what memory is*, the optional **Physical Memory Protection (PMP)** unit defines *who can access it*. It's a configurable hardware firewall that enforces access permissions (read, write, execute) for different memory regions.

The PMP is configured using dedicated M-mode CSRs and is powerful enough to:
* **Grant** permissions to less-privileged modes (e.g., allow Supervisor or User mode to access a specific RAM region).
* **Revoke** permissions from M-mode itself, creating a secure sandbox for trusted code.

Each PMP entry defines a memory region using two types of CSRs:

* **`pmpaddrZ`:** A register holding the physical address for a region.
* **`pmpZcfg`:** An 8-bit configuration field that sets the rules for that region. These small fields are packed together into larger CSRs (e.g., `pmpcfg0`).

The key configuration bits in `pmpZcfg` are:
* **`R` / `W` / `X`:** Grant **R**ead, **W**rite, or e**X**ecute permissions if set.
* **`A` (Address Matching):** Determines the region's size. Common modes are `TOR` (Top of Range), where two `pmpaddr` registers define the region's boundaries, and `NAPOT` (Naturally Aligned Power-of-Two), where the address and its lower bits define a power-of-two-sized region.
* **`L` (Lock):** When set, this bit prevents any further writes to the region's configuration CSRs until the next hart reset. This is a security feature to prevent tampering.