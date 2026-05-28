# MMU-SMMU Study

A structured study repo for learning MMU, virtual memory, ARM A-series memory management, and related translation concepts.

## Index

- [Course Overview](#course-overview)
- [Module Guide](#module-guide)
- [High-Level Diagrams](#high-level-diagrams)
- [Repository Structure](#repository-structure)
- [Suggested Study Flow](#suggested-study-flow)

## Course Overview

This repository is designed as a step-by-step study path for understanding how memory management works in modern ARM A-series systems. It starts from the motivation for memory management, moves through virtual memory and MMU mappings, and then builds toward ARMv8/AArch64 translation structures and a full numerical address-translation example.

## Module Guide

### [Module 1: Foundations](course/01-foundations.md)
Learn why memory management is needed, why ARM A-series processors require an MMU, and where the MMU logically fits in the CPU-to-memory path.

### [Module 2: Virtual Memory Basics](course/02-virtual-memory.md)
Understand the virtual memory assumption, the difference between logical, virtual, and physical addresses, and how isolation and protection are achieved.

### [Module 3: MMU Mappings](course/03-mmu-mappings.md)
Study what MMU mappings are, where translation information is stored, how the MMU retrieves it, and how the TLB and page-table walk work together.

### [Module 4: ARM Translation Structures](course/04-arm-translation-structures.md)
Explore ARM translation table entry formats, the relationship between TTBR, TLB, and page-table walks, and the difference between block and page mappings.

### [Module 5: ARMv8 / AArch64 Translation](course/05-armv8-aarch64.md)
Dive into AArch64 translation, TTBR0_EL1 and TTBR1_EL1, TCR_EL1, MAIR_EL1, multi-level translation tables, and descriptor types.

### [Module 6: Numerical Translation Example](course/06-numerical-translation-example.md)
Follow a complete step-by-step AArch64 numerical translation example and see how offsets and final physical addresses are formed.

## High-Level Diagrams

### 1. CPU to Memory Access Path

```text
Program / Instruction
        |
        v
       CPU
        |
        v
   Virtual Address
        |
        v
       MMU
     /     \
TLB hit   TLB miss
  |          |
  |      Page Table Walk
  |          |
  +----------+
        |
        v
 Physical Address
        |
        v
   RAM / Device Memory
```

### 2. Virtual Memory Abstraction

```text
Process View:
+-----------------------------+
| Code                        |
| Data                        |
| Heap                        |
| ...                         |
| Stack                       |
+-----------------------------+
      looks contiguous

Physical Reality:
+--------+   +--------+   +--------+   +--------+
| FrameA |   | FrameB |   | FrameC |   | FrameD |
+--------+   +--------+   +--------+   +--------+
   ^             ^            ^            ^
   |_____________|____________|____________|
        mapped through page tables
```

### 3. Page Table Walk Flow

```text
Virtual Address
      |
      +--> L0 index --> Level 0 table entry
      |
      +--> L1 index --> Level 1 table entry
      |
      +--> L2 index --> Level 2 table entry
      |
      +--> L3 index --> Level 3/page entry
      |
      +--> page offset
                    |
                    v
      Physical Page Base + Offset
                    |
                    v
             Physical Address
```

### 4. TLB vs Page Table

```text
+---------------------------+
| TLB                       |
| Small, fast cache         |
| Stores recent mappings    |
+---------------------------+
            |
    hit --> use mapping
    miss --> walk page tables in RAM
            |
            v
+---------------------------+
| Page Tables in Memory     |
| Full translation database |
| Maintained by OS          |
+---------------------------+
```

## Repository Structure

- `README.md` — overview, module guide, study path, and diagrams
- `SUMMARY.md` — quick course navigation
- `course/01-foundations.md` — fundamentals of memory management and MMU need
- `course/02-virtual-memory.md` — virtual memory concepts and assumptions
- `course/03-mmu-mappings.md` — translation mappings, TLB, and page-table walks
- `course/04-arm-translation-structures.md` — ARM translation entries and mapping structure
- `course/05-armv8-aarch64.md` — ARMv8/AArch64 translation details
- `course/06-numerical-translation-example.md` — worked address-translation example

## Suggested Study Flow

1. Start with why memory management is required.
2. Understand virtual memory as an abstraction layer.
3. Learn how MMU mappings are stored and retrieved.
4. Study ARM translation structures and descriptors.
5. Move to ARMv8/AArch64 translation control and table hierarchy.
6. Finish with the numerical translation walkthrough.
