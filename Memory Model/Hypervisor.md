## Hypervisor Extension (H-extension)

The Hypervisor extension (`H` in the `misa` register) adds support for hardware virtualization. Instead of being a separate privilege level, it extends Supervisor mode to allow a **hypervisor** to manage one or more **guest operating systems**.

This extension effectively splits S-mode into two:
* **HS-mode (Hypervisor-extended Supervisor):** The mode the hypervisor itself runs in.
* **VS-mode (Virtual Supervisor):** The mode a guest OS runs in. It believes it is running in S-mode.
* **VU-mode (Virtual User):** The mode that applications running on the guest OS use.

### Hypervisor CSRs

To manage virtualization, the H-extension introduces two new sets of CSRs: one for the hypervisor (HS-mode) and one for the virtualized guest (VS-mode).

* **Hypervisor-level CSRs (`h` prefix):** Used by the hypervisor to manage the virtual machine. Examples include `hstatus`, `hip`, `hie`, and `hgatp`. The `hgatp` register is crucial, as it controls the translation from a guest's physical addresses to the machine's actual physical addresses (the second stage of translation).
* **Virtual Supervisor CSRs (`vs` prefix):** These are virtualized versions of the S-mode CSRs, accessible to the guest OS. The guest interacts with `vsstatus`, `vsie`, `vsip`, and `vsatp` just as a normal OS would interact with `sstatus`, etc.

### Key Hypervisor Instructions

* **`HLV.*` / `HSV.*` (Hypervisor Load/Store):** A set of memory access instructions (e.g., `HLV.W`, `HSV.B`) used by the hypervisor to explicitly read from or write to the guest's memory, applying the two-stage address translation managed by `vsatp` and `hgatp`.
* **`HLVX.*` (Hypervisor Load Executable):** Special load instructions that check for execute permissions instead of read permissions during address translation.
* **`HFENCE.VVMA` (Hypervisor Fence, Virtual VMA):** A fence for the *first stage* of address translation (guest virtual to guest physical), which is controlled by the `vsatp` CSR.
* **`HFENCE.GVMA` (Hypervisor Fence, Guest VMA):** A fence for the *second stage* of address translation (guest physical to machine physical), which is controlled by the `hgatp` CSR.
