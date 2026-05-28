# MMU-SMMU Study

A structured study repo for learning MMU, virtual memory, ARM A-series memory management, and related translation concepts.

## Course Outline

### Module 1: Foundations
- Why memory management is needed
- Why MMU is needed in ARM A-series CPUs
- Logical placement of MMU hardware

### Module 2: Virtual Memory Basics
- Virtual memory assumption
- Logical vs virtual vs physical address
- Process isolation and protection

### Module 3: MMU Mappings
- What MMU mappings are
- Where mappings are stored
- How mappings are retrieved
- TLB and page-table walks

### Module 4: ARM Translation Structures
- ARM translation table entry format
- TTBR, TLB, and page table walk in ARM
- Section/block mapping vs page mapping

### Module 5: ARMv8 / AArch64 Translation
- AArch64 translation overview
- TTBR0_EL1 and TTBR1_EL1
- TCR_EL1 and MAIR_EL1
- Multi-level translation tables
- Block descriptors and page descriptors

### Module 6: Numerical Translation Example
- Step-by-step address translation in AArch64
- Offset handling
- Physical address formation

## Repository Structure
- `README.md` — overview and course outline
- `course/01-foundations.md`
- `course/02-virtual-memory.md`
- `course/03-mmu-mappings.md`
- `course/04-arm-translation-structures.md`
- `course/05-armv8-aarch64.md`
- `course/06-numerical-translation-example.md`

## Suggested Study Flow
1. Start with memory-management motivation.
2. Understand virtual memory as an abstraction.
3. Learn how MMU mappings are stored and used.
4. Study ARM translation structures.
5. Move to ARMv8/AArch64-specific details.
6. Finish with the numerical walk-through example.
