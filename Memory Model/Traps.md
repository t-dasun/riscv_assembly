## Exceptions, Interrupts, Traps & System Calls

* Execution cannot complete in current privilege, an **exception** is raised and it is routed into a so-called **trap**, which simply means a control transfer to a trap handler, possibly running at a higher privilege level.
* **Exceptions** are **synchronous**, **interrupts** are **asynchronous**.
* There are 4 types of traps:

    * **Contained Trap** - Handled by the software that raises it. Execution resumes after resolution.
    * **Requested Trap** - An explicit request is done by software to the execution environment via the ECALL instruction (e.g., **System calls**).
    * **Invisible Trap** - A trap raised and handled by the execution environment that is not visible to the software running inside it.
    * **Fatal Trap** - Causes the execution environment to stop execution. How execution is stopped is defined by the given execution environment.

---

## Privileged ISA and CSRs

* **CSRs** are accessed using a 12-bit address. This 12-bit address is not just a location; it's designed to encode specific properties of the register, making access faster and more secure.
* The RISC-V Privileged ISA divides the CSR register file according to read/write (R/W) and privilege accessibility requirements. This information is directly encoded in the CSR address.
    * The two most significant bits encode whether the register is read/write (00, 01, or 10) or read-only (11).
    * The next two bits encode the lowest privilege level that can access the CSR (00 for unprivileged, 01 for Supervisor, 10 for Hypervisor, 11 for Machine-level CSRs).

* **WARL (Write-Any, Read-Legal)**
    * This field lets you attempt to write any value to it, but the hardware will only accept and store a legal one.
    * In short: You can try to write whatever you want, but the hardware cleans it up and forces it into a supported value. Think of it like a volume knob that only has settings for 0, 5, and 10. If you try to set it to 7, it might just snap to 5.

* **WLRL (Write-Legal, Read-Legal)**
    * This field requires you to write a legal value. Writing an unsupported value may cause unpredictable behavior.
    * In short: You must only write values that the hardware explicitly supports. It’s like a gear shift in a car—you can only put it in Park, Reverse, or Drive, not somewhere in between.

* **WPRI (Write-Preserve, Read-Ignore)**
    * These fields are reserved for future use and should be ignored by software.
    * In short: When writing, preserve whatever value is already there (write them as 0). When reading, ignore whatever value you see. It's like a section on a form marked "For office use only"—just leave it alone.