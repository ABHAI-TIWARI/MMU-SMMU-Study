# Module 5: ARMv8 / AArch64 Translation

This module explains ARMv8-A running in AArch64, the main translation registers, multi-level translation tables, and the important descriptor and control fields used during memory translation.

---

## 1. ARMv8 / AArch64 Translation Overview

In **ARMv8-A running AArch64**, virtual address translation is typically done with **multi-level translation tables**, often using **4 KB pages** and commonly **4 levels** of tables.

The exact number of levels depends on the configured virtual address size, but a common teaching model is:

- Level 0
- Level 1
- Level 2
- Level 3

### Basic idea

A virtual address is split into fields:

- bits for indexing Level 0 table
- bits for indexing Level 1 table
- bits for indexing Level 2 table
- bits for indexing Level 3 table
- final page offset

For a common **4 KB page** system:

- page offset = **12 bits**
- each table index often uses **9 bits**
- each table has **512 entries** because `2^9 = 512`

So conceptually:

```text
Virtual Address = [L0 index][L1 index][L2 index][L3 index][page offset]
```

---

## 2. TTBR0_EL1 and TTBR1_EL1

In AArch64, the operating system often uses:

- **TTBR0_EL1**
- **TTBR1_EL1**

These are **translation table base registers**.

### Typical usage

Very commonly:

- **TTBR0_EL1**: used for lower virtual addresses, often user space
- **TTBR1_EL1**: used for higher virtual addresses, often kernel space

So depending on the virtual address range, the MMU chooses which TTBR to use as the starting point.

### Why these registers matter

Because translation tables are in RAM, and the MMU must know where to begin the translation walk.

So:

- the operating system creates translation tables in RAM
- the operating system writes the base addresses into TTBR0_EL1 and TTBR1_EL1
- the MMU uses the selected TTBR as the root of the translation-table hierarchy

---

## 3. TCR_EL1

**TCR_EL1** means **Translation Control Register at EL1**.

This register controls **how address translation behaves**.

It defines things such as:

- virtual address size
- granule size (for example 4 KB)
- how the virtual address space is split between TTBR0 and TTBR1
- shareability and cacheability properties for translation table walks
- size of the region handled by each TTBR

### Why it matters

Without TCR_EL1, the MMU would not know:

- how many bits of the virtual address are valid
- how many table levels are needed
- what granule size is used
- how to interpret the address-space split between user and kernel regions

So TCR_EL1 defines the **rules of the translation system**.

---

## 4. MAIR_EL1

**MAIR_EL1** means **Memory Attribute Indirection Register at EL1**.

It defines memory behavior such as:

- normal memory
- device memory
- cacheable memory
- non-cacheable memory
- write-back or write-through behavior depending on system setup

### How it is used

Page-table entries often store an **attribute index**, not the full memory-type description.

That index selects one of the attribute encodings stored in MAIR_EL1.

### Example idea

A page-table entry may say:

- `AttrIndx = 2`

Then MAIR_EL1 entry 2 may define that as:

- normal write-back cacheable memory

Another index may define:

- device memory

So MAIR_EL1 acts like a **lookup table for memory types**.

---

## 5. Translation Tables

The translation tables are the actual mapping structures stored in RAM.

The operating system creates and manages them.

Each entry in a table may:

- point to another table
- or directly map a block/page

These tables are arranged in a hierarchy such as:

- Level 0 table
- Level 1 table
- Level 2 table
- Level 3 table

### Why multiple levels are used

Because a single flat table for the whole virtual address space would be extremely large.

Multi-level tables allow memory to be allocated only where needed.

---

## 6. Table Descriptor

A **table descriptor** is an entry that points to the **next-level translation table**.

It does **not** directly provide the final physical page mapping.

### Example

A Level 0 entry may say:

- go to Level 1 table at physical address `0x81000000`

So the MMU reads that table next.

### Meaning

A table descriptor basically says:

> Translation continues in another table.

---

## 7. Block Descriptor

A **block descriptor** directly maps a **large memory region**.

Instead of going to another lower-level table, the MMU can stop earlier and use that mapping.

### Why block mappings are useful

- fewer table levels to walk
- fewer entries needed
- useful for large continuous regions

### Limitation

A block applies one set of rules to the whole mapped region, so it has less fine-grained control than pages.

A block descriptor can appear at intermediate levels such as Level 1 or Level 2, depending on the translation configuration.

---

## 8. Page Descriptor

A **page descriptor** is the final descriptor that maps a **page**, usually at the last level.

It gives:

- physical page base address
- access permissions
- execute permissions
- memory attributes
- access flag and other control bits

For a 4 KB granule system, the final page size is commonly **4 KB**.

This is the most fine-grained mapping level.

---

## 9. Level 0 Table

This is the top-level translation table.

After selecting TTBR0_EL1 or TTBR1_EL1, the MMU begins here.

The Level 0 index bits from the virtual address select one entry in this table.

That entry usually:

- points to a Level 1 table
- or is invalid

So Level 0 is the first directory of the translation hierarchy.

---

## 10. Level 1 Table

The Level 1 table is reached through a Level 0 table descriptor.

The MMU uses the Level 1 index bits from the virtual address to choose an entry.

That entry may:

- point to a Level 2 table
- be a block descriptor
- be invalid

So Level 1 is the second stage of lookup.

---

## 11. Level 2 Table

The Level 2 table is reached if the Level 1 entry points to another table.

The Level 2 index bits select an entry.

That entry may:

- point to a Level 3 table
- directly map a block
- be invalid

So this level refines the mapping further.

---

## 12. Level 3 Table

The Level 3 table is often the final level for a **4 KB granule** walk.

The Level 3 index selects the final page descriptor.

That descriptor gives:

- physical page base address
- access permissions
- execute permissions
- memory attributes

After that, the MMU combines the result with the page offset.

---

## 13. Page Offset

The **page offset** is the lowest part of the virtual address.

For a **4 KB page**:

- page size = 4096 bytes
- offset bits = 12

This offset is **not translated**.

It is copied directly into the final physical address.

### Why?

Because translation changes the **page base**, not the byte position inside the page.

### Example

- physical page base = `0x45603000`
- offset = `0xABC`

Final physical address:

```text
0x45603000 + 0xABC = 0x45603ABC
```

---

## 14. TLB

**TLB** means **Translation Lookaside Buffer**.

This is a small, fast cache inside the CPU that stores recently used virtual-to-physical mappings.

Before walking page tables, the MMU checks the TLB.

### If TLB hit

- translation is found quickly
- no page-table walk is needed

### If TLB miss

- the MMU reads translation tables from memory
- performs the walk
- then fills the TLB with the result

The TLB is critical for performance.

---

## 15. Page Table Walk

A **page table walk** happens when the TLB does not contain the needed translation.

The MMU then:

1. starts from TTBR0_EL1 or TTBR1_EL1
2. reads the Level 0 entry
3. goes to Level 1 if needed
4. then Level 2
5. then Level 3 if needed
6. gets the final mapping
7. checks permissions and attributes
8. forms the physical address
9. stores the result in the TLB

This is slower than a TLB hit because it requires reading translation tables from memory.

---

## 16. Access Permissions

Page and block descriptors contain access control information.

This determines whether memory can be:

- read
- written
- executed
- accessed from user mode
- accessed only from privileged/kernel mode

If access violates the descriptor permissions, the MMU raises a fault.

This is a key part of process isolation and operating-system protection.

---

## 17. Execute Control: UXN and PXN

ARMv8 uses execution-control bits such as:

- **UXN** = Unprivileged Execute Never
- **PXN** = Privileged Execute Never

These decide whether code execution is allowed from that memory region.

### Example

A page may be:

- readable
- writable
- but not executable

This improves security by preventing data pages from being executed as code.

---

## 18. AttrIndx

**AttrIndx** is the attribute-index field inside a block or page descriptor.

It does not itself fully describe the memory type. Instead, it points to a memory-attribute encoding in **MAIR_EL1**.

So:

- descriptor says `AttrIndx = 1`
- MAIR_EL1 entry 1 defines what that means

This avoids repeating full memory-type encodings in every descriptor.

---

## 19. AF — Access Flag

**AF** means **Access Flag**.

It indicates whether the page has been accessed.

The MMU may require this flag to be set for normal access behavior, depending on configuration.

This helps the operating system track page usage.

---

## 20. Physical Address Formation

After the MMU finds the final descriptor, it gets:

- physical base address of the block or page

Then it adds the unchanged offset from the virtual address.

So:

```text
Physical Address = Physical Page Base + Offset
```

That gives the actual memory-system address used to access RAM or devices.

---

## 21. Step-by-Step Translation Flow

Here is the full translation flow in simple order:

1. program generates virtual address
2. MMU checks TLB
3. if miss, select TTBR0_EL1 or TTBR1_EL1
4. use Level 0 index to find entry
5. go to Level 1
6. go to Level 2
7. go to Level 3 or stop at a block
8. obtain physical base address
9. check permissions and attributes
10. add page offset
11. produce physical address
12. access memory
13. cache the translation in the TLB

---

## 22. Short Summary

### TTBR0_EL1 / TTBR1_EL1
Tell the MMU where the translation-table trees begin.

### TCR_EL1
Controls translation rules.

### MAIR_EL1
Defines memory types.

### Translation tables
Store mappings in RAM.

### Table descriptor
Points to the next-level table.

### Block descriptor
Maps a large region.

### Page descriptor
Maps a final page.

### TLB
Caches recent translations.

### Page table walk
Finds mappings from RAM on a TLB miss.

### Offset
Is copied unchanged into the final physical address.
