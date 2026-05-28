# Module 3: MMU Mappings

This module explains what MMU mappings are, where they are stored, how the MMU retrieves them, and how the TLB and page-table walk work together during address translation.

---

## 1. What Are MMU Mappings?

MMU mappings are the **translation rules** that tell the CPU how a **virtual address** becomes a **physical address**.

A mapping says things like:

- virtual page -> physical frame
- allowed access:
  - read / write
  - execute / no-execute
  - user / privileged
- memory type:
  - normal memory
  - device memory
- cache and shareability attributes

So a mapping is not just an address conversion. It also includes **protection information** and **memory attributes**.

---

## 2. Granularity of Mapping

The MMU usually maps memory in **pages or blocks**, not byte-by-byte.

### Basic idea

- Virtual address = **virtual page number + offset**
- Physical address = **physical frame number + offset**

Example:

- Virtual address = `0x12345678`
- Page size = `4 KB`
- Virtual page = `0x12345`
- Offset = `0x678`

If the page table says:

- virtual page `0x12345` -> physical frame `0x8ABCD`

then:

- physical address = `0x8ABCD678`

The offset stays the same. Only the page/frame part changes.

---

## 3. Where MMU Mappings Are Stored

### Main storage: page tables in memory

The main mapping database is stored in **page tables** or **translation tables** in RAM.

These tables are created and maintained by the operating system.

In ARM A-series processors, these are often called:

- translation tables
- page tables
- descriptor tables

They live in normal memory, not inside the program.

---

## 4. How the MMU Knows Where the Tables Are

The CPU has special control registers that point to the base of the translation tables.

Conceptually:

- the operating system creates translation tables in RAM
- the operating system writes the base address into MMU-related registers
- the MMU uses those registers as the starting point for translation

So the MMU does not search randomly. It starts from a known table base.

---

## 5. Where Mappings Are Retrieved From During Execution

There are two levels of retrieval:

### A. Fast path: TLB

The MMU first checks the **TLB (Translation Lookaside Buffer)**.

The TLB is a small, fast cache inside the processor that stores recently used mappings.

If the needed mapping is found there:

- translation is immediate
- no page-table memory access is needed

This is called a **TLB hit**.

### B. Slow path: page table walk

If the mapping is not in the TLB:

- the MMU performs a **page-table walk**
- it reads translation-table entries from RAM
- if the mapping is valid, it loads the result into the TLB
- then it completes the access

This is called a **TLB miss**.

So retrieval order is:

```text
TLB -> page tables in RAM -> update TLB
```

---

## 6. What Is Inside a Page-Table Entry?

A page-table entry typically contains:

- physical frame number or output address
- valid bit
- access permissions
- execute-never information
- memory attribute index or cacheability info
- shareability
- access flag
- sometimes dirty/modified state depending on architecture or OS design

So one entry tells the system both:

- **where** the memory is
- **how** it may be used

---

## 7. Step-by-Step Mapping Lookup

Suppose a program accesses virtual address `0x40001234`.

### Step 1: CPU generates virtual address
The load/store unit or instruction fetch unit generates the virtual address.

### Step 2: MMU checks TLB
- If entry exists: use it
- If not: continue to table walk

### Step 3: MMU reads page tables from memory
Using the translation table base register, the MMU walks the page tables.

### Step 4: MMU finds matching descriptor
It gets:
- physical base address
- attributes
- permissions

### Step 5: MMU combines physical frame with offset
Example:

- page base -> `0x90001000`
- offset -> `0x234`

Result:

- physical address = `0x90001234`

### Step 6: MMU checks permission
If access is allowed:
- memory access continues

If not:
- exception or fault is raised

### Step 7: TLB is updated
The found translation is usually cached in the TLB for future accesses.

---

## 8. Multi-Level Page Tables

Modern ARM A-series CPUs usually use **multi-level translation tables**.

### Why multi-level tables are used
Because a single flat table for all virtual addresses would be too large.

So translation is done in levels, for example:

- Level 1 entry points to Level 2 table
- Level 2 points to Level 3
- Final level gives the actual page mapping

This reduces memory usage because only needed lower-level tables must exist.

---

## 9. Who Creates and Updates Mappings?

The **operating system kernel** creates the mappings.

It maps:

- kernel code and data
- user processes
- stack and heap
- shared libraries
- memory-mapped files
- device registers

When a process switch happens, the OS may change which page tables are active.

That means the same virtual address in two processes can map differently.

---

## 10. Are Mappings Stored Only in RAM?

Not only in RAM.

Mappings conceptually exist in two places:

### Permanent or authoritative copy
- page tables in RAM

### Cached copy
- TLB inside the CPU

So if someone asks where MMU mappings are stored, the best answer is:

- **stored authoritatively in page tables in memory**
- **cached in the TLB for fast retrieval**

---

## 11. What Happens If a Mapping Is Missing?

If the page-table walk finds no valid mapping:

- the MMU raises a fault

Examples include:

- translation fault
- page fault
- access fault

Then the operating system may:

- allocate a page
- load data
- reject access
- terminate the process if the access is invalid

---

## 12. Simple Analogy

Think of virtual memory like hotel room numbers.

- Guest says: “Take me to room 305” -> virtual address
- Front desk directory says where that room actually is -> page table
- Concierge remembers common rooms quickly -> TLB

So:

- **page table** = full directory
- **TLB** = quick memory/cache of recent lookups

---

## 13. Short Summary

**MMU mappings are virtual-to-physical translation entries, stored mainly in page tables in RAM, cached in the TLB, and retrieved by checking the TLB first and walking the page tables on a miss.**

### Very short summary

- **What are mappings?** Virtual page -> physical frame + permissions/attributes
- **Where stored?** Page tables in RAM
- **Where cached?** TLB in CPU
- **How retrieved?** TLB lookup first, then page-table walk from memory
