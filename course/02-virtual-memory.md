# Module 2: Virtual Memory Basics

This module explains the virtual memory assumption, why software works with virtual addresses instead of physical addresses, and how the MMU and operating system make that abstraction appear real.

---

## 1. What Is the Virtual Memory Assumption?

The **virtual memory assumption** means software is written as if it is using a **large, private, contiguous memory space**, even though the actual physical RAM may be smaller, fragmented, or shared with other programs.

### Core idea

A program assumes:

- the addresses it uses are its own **logical/virtual addresses**
- memory looks continuous
- it does not need to know where data is physically stored in RAM
- it cannot directly see other processes’ memory

The hardware and operating system make this assumption appear true by using the **MMU** and **page tables**.

---

## 2. Why This Assumption Exists

Without virtual memory, every program would need to know:

- exact physical RAM locations
- where other programs are loaded
- which memory regions are free
- how to avoid collisions with the OS and devices

That would make multitasking, protection, relocation, and modern OS features very difficult.

So instead, the system creates an abstraction:

> Each process gets its own address space.

That is the virtual memory assumption.

---

## 3. What a Process Assumes

When a process runs, it behaves as though:

1. **It owns the address space**
   - It may use addresses like `0x400000` or high stack addresses.
   - These are virtual addresses, not direct RAM locations.

2. **Its memory is contiguous**
   - Code, heap, stack, and libraries appear arranged in a clean logical layout.

3. **It has more usable memory than immediate RAM layout suggests**
   - Because the OS can map pages flexibly and even move inactive pages to secondary storage in some systems.

4. **Its memory is protected**
   - Other processes cannot simply read or write it.

5. **The same program can run anywhere in physical memory**
   - Because virtual addresses stay consistent even if physical placement changes.

---

## 4. How the Assumption Is Implemented

### Step 1: CPU generates a virtual address
A running instruction accesses memory using a virtual address.

Example:
- load from virtual address `0x1000`

### Step 2: MMU translates it
The MMU checks page tables and converts that virtual address into a physical address.

Example:
- virtual `0x1000` -> physical `0xA0001000`

### Step 3: Permissions are checked
The MMU also checks:
- is this page readable?
- writable?
- executable?
- user-accessible or kernel-only?

### Step 4: Access proceeds or faults
- If valid: memory access happens
- If invalid: page fault, translation fault, or permission fault occurs

---

## 5. Important Assumptions in Virtual Memory

### A. Private address space assumption
Each process assumes its address space is private.

Even if two processes both use virtual address `0x4000`, they may map to different physical pages.

Example:
- Process A: `0x4000` -> physical `0x9000`
- Process B: `0x4000` -> physical `0xD000`

Same virtual address, different actual memory.

### B. Contiguous memory assumption
A process assumes memory is continuous.

But physically, its pages may be scattered:

- Virtual page 0 -> physical frame 25
- Virtual page 1 -> physical frame 103
- Virtual page 2 -> physical frame 8

To the program, it still looks like one continuous region.

### C. Large memory assumption
A process assumes it has access to a large address range, often much larger than the currently installed free RAM.

This is possible because:
- only needed pages are mapped
- some pages may be shared
- inactive pages may be moved out of RAM in some systems

### D. Location independence assumption
Programs assume they can run without caring where they are placed in physical RAM.

This supports:
- relocation
- dynamic loading
- shared libraries
- process creation

### E. Protection assumption
Programs assume inaccessible memory stays inaccessible.

For example:
- app memory is isolated from kernel memory
- read-only code pages cannot be overwritten
- non-executable pages cannot be executed

This is enforced by the MMU.

---

## 6. Why It Is Called an “Assumption”

It is called an assumption because the program behaves **as if** the virtual address space is the real memory layout.

But in reality:

- addresses are translated
- memory may be non-contiguous
- pages may be absent
- data may be shared or swapped
- permissions may differ by region

So virtual memory is an **abstraction** presented to software.

---

## 7. Example Layout

Suppose a program thinks it has this memory layout:

```text
0x00000000 - 0x0000FFFF   code
0x00010000 - 0x0001FFFF   data
0x00020000 - 0x0002FFFF   heap
0x7FFF0000 - 0x7FFFFFFF   stack
```

This looks neat and contiguous.

But actual RAM placement may be:

```text
code  -> physical frames 12, 44, 90
data  -> physical frames 7, 130
heap  -> physical frames 201, 5, 66
stack -> physical frames 18, 19
```

The program never needs to know this. That is the virtual memory assumption in action.

---

## 8. Benefits of This Assumption

1. **Simpler programming model**
   - Programs use normal addresses and do not manage physical RAM directly.

2. **Process isolation**
   - Each process gets its own address space.

3. **Better memory utilization**
   - Physical pages can be allocated wherever free space exists.

4. **Supports multitasking**
   - Many processes can coexist safely.

5. **Supports advanced OS features**
   - demand paging
   - copy-on-write
   - shared libraries
   - memory-mapped files
   - fork/exec model
   - ASLR

---

## 9. In ARM A-Series Context

In ARM Cortex-A and other A-series processors:

- the CPU generates **virtual addresses**
- the **MMU** translates them using translation tables
- the system supports:
  - user/kernel separation
  - page permissions
  - memory attributes
  - shared memory mappings
  - virtual-to-physical remapping

Because ARM A-series targets rich operating systems, this virtual memory assumption is fundamental.

---

## 10. Relation to Logical Address

Textbooks often use the terms:

- **logical address**
- **virtual address**

These are often used similarly in this context.

So the assumption is also:

> The process works with logical addresses, not physical addresses.

---

## 11. What Happens If the Assumption Breaks

If a process accesses a virtual address that:

- is not mapped
- lacks permission
- is invalid

then the MMU raises an exception such as:

- page fault
- translation fault
- access fault
- permission fault

So the virtual memory model is maintained only for valid mappings.

---

## 12. Short Summary

**Virtual memory assumption** means a process behaves as if it has its own large, continuous, protected memory space, while the OS and MMU secretly translate and manage that space over real physical memory.

### One-line definition

**It is the assumption that the addresses used by a program are part of a private logical address space, not direct physical RAM addresses.**
