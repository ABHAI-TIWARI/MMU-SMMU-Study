# Module 4: ARM Translation Structures

This module explains ARM translation table entry format, the relationship between TTBR, TLB, and page-table walks, and the difference between section/block mapping and page mapping.

---

## 1. ARM Translation Table Entry Format

A **translation table entry** is one record inside a page table. It tells the MMU:

- whether the entry is valid
- where the next table is, or where the physical page/block is
- what permissions apply
- what memory attributes apply

### What an entry contains

A typical ARM translation entry contains fields such as:

- **Valid bit**
  - tells whether the entry can be used

- **Type bits**
  - tells whether the entry points to:
    - a next-level table, or
    - a final mapped block/page

- **Output address**
  - physical base address of the block or page

- **Access permissions**
  - read/write control
  - privileged/user access control

- **Execute control**
  - executable or execute-never

- **Access flag**
  - indicates that the page has been accessed

- **Shareability**
  - whether memory is non-shareable, inner-shareable, or outer-shareable

- **Attribute index / memory type**
  - selects device memory or normal memory behavior through attribute registers

### Two broad kinds of entries

#### A. Table descriptor
This entry does **not** directly map memory. Instead, it points to the **next-level page table**.

Example idea:
- Level 1 entry -> address of Level 2 table

#### B. Block/Page descriptor
This entry gives the **actual mapping**:
- virtual range -> physical range

If it is a higher-level final mapping, it may map a **block**.
If it is the last level, it may map a **page**.

### Simple conceptual example

Suppose one entry says:

- valid = 1
- type = page
- output address = `0x80000000`
- read/write = allowed
- user access = allowed
- execute = no
- memory type = normal cacheable

That means the corresponding virtual page maps to physical memory near `0x80000000` with those permissions.

### Key idea

A translation table entry is basically:

**For this virtual region, go here in physical memory, and use these protection/attribute rules.**

---

## 2. TTBR, TLB, and Page Table Walk in ARM

This is the actual mechanism ARM uses to perform translation.

### A. TTBR

**TTBR** means **Translation Table Base Register**.

It stores the **base address of the translation table** in memory.

So the MMU uses TTBR as the starting point to find page table entries.

You can think of TTBR as:

- Where does the page-table hierarchy begin?

In ARM systems, the operating system sets this register when enabling or changing address spaces.

#### Why TTBR is needed
Because page tables are in RAM, and the MMU must know where they are.

So:

- OS creates page tables in RAM
- OS writes their base into TTBR
- MMU starts translation from TTBR

### B. TLB

**TLB** means **Translation Lookaside Buffer**.

It is a small cache inside the processor that stores recently used address translations.

Instead of walking the page tables every time, the MMU first checks the TLB.

#### If TLB hit
If the mapping is already present:

- translation is fast
- access continues immediately

#### If TLB miss
If the mapping is not present:

- MMU performs a page-table walk
- gets the translation from RAM
- stores it in the TLB
- retries or completes the access

So the TLB improves performance significantly.

### C. Page table walk

A **page table walk** is the process of reading translation-table entries from memory to resolve a virtual address.

#### Step-by-step

Suppose CPU wants virtual address `VA`.

1. **CPU issues memory access**
   - instruction fetch or data access generates `VA`

2. **MMU checks TLB**
   - if hit -> done
   - if miss -> walk tables

3. **MMU uses TTBR**
   - TTBR gives base address of first-level table

4. **MMU uses bits of VA**
   - different parts of the virtual address index different table levels

5. **Read level-1 entry**
   - may point to next-level table
   - or may directly map a block

6. **If needed, read next level**
   - continue until final mapping is found

7. **Obtain physical base address**
   - combine with page offset from VA

8. **Check permissions and attributes**
   - read/write/execute, privilege, memory type, etc.

9. **Fill TLB**
   - cache result for future use

10. **Complete memory access**

### Simple flow

```text
Virtual Address
      |
      v
   Check TLB
   /      \
 hit      miss
 |          |
use entry   use TTBR -> walk page tables in RAM
 |                          |
 v                          v
physical address <---- found translation
      |
      v
memory access
```

### Main relationship

- **TTBR** = where page tables start
- **TLB** = fast cache of translations
- **Page table walk** = slow lookup in RAM when the TLB misses

---

## 3. Difference Between Section/Block Mapping and Page Mapping

This is about the **size of the memory region** being mapped.

Historically in ARM terminology, people often say:

- **section mapping**
- **page mapping**

In newer terminology, section-like mappings are often called **block mappings**.

### A. Section/Block mapping

A **section** or **block mapping** maps a **large continuous memory region** with one entry.

Older ARM systems often used section mappings such as **1 MB** sections.
In ARMv8-style terminology, larger mappings are commonly called **blocks**.

#### Characteristics

- large region
- fewer table entries
- faster or simpler management
- less flexible

#### Good for

- kernel linear mapping
- large device regions
- simple memory layout
- boot-time setup

#### Limitation

Because one large region is mapped with one descriptor:

- permissions are the same for the entire region
- memory attributes are the same for the entire region
- fine-grained control is not possible inside that region

### B. Page mapping

A **page mapping** maps a **small memory unit**, such as a **4 KB** page.

#### Characteristics

- smaller granularity
- more flexible
- more table entries needed
- better control

#### Good for

- user processes
- heap/stack/code separation
- fine-grained permissions
- demand paging
- copy-on-write

#### Advantage

Each small page can have different:

- physical location
- read/write permission
- execute permission
- caching attribute

### Example difference

Imagine **1 MB** of virtual memory.

#### Using section/block mapping
- one entry maps the whole 1 MB region

#### Using page mapping
- if page size is 4 KB:
- `1 MB / 4 KB = 256 pages`
- so 256 separate entries may be used

Now each 4 KB region can be controlled independently.

### Tradeoff

#### Section/block mapping
**Pros**
- simpler
- fewer entries
- lower table overhead
- useful for large fixed regions

**Cons**
- coarse-grained
- poor flexibility

#### Page mapping
**Pros**
- fine-grained control
- better protection
- better support for sharing and paging

**Cons**
- larger tables
- more management overhead

### Practical intuition

Use **section/block mapping** when:

- a large region can be treated uniformly

Use **page mapping** when:

- each small region may need different rules

---

## 4. Short Summary

### Translation table entry format
A translation entry stores:
- valid/type bits
- physical output address
- permissions
- execute control
- memory attributes

### TTBR, TLB, and page table walk
- **TTBR** points to page tables in RAM
- **TLB** caches recent mappings
- **Page table walk** happens on a TLB miss

### Section/block mapping vs page mapping
- **section/block mapping** = large region, fewer entries, less flexible
- **page mapping** = small region, more entries, more flexible
