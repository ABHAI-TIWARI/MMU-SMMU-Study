# Module 6: Numerical Translation Example

This module gives a simple numerical walk-through of AArch64 address translation using a multi-level page table structure and shows how the final physical address is formed.

---

## 1. Goal of the Example

We want to understand how the MMU translates a **virtual address** into a **physical address** step by step.

We will use a simplified teaching model based on:

- ARMv8-A / AArch64
- 4 KB pages
- 4 translation levels
- 9-bit index per level
- 12-bit page offset

### Conceptual split

```text
VA = [L0][L1][L2][L3][offset]
```

Where:

- each level index = 9 bits
- offset = 12 bits

---

## 2. Example Virtual Address

Suppose the program accesses this virtual address:

```text
VA = 0x0000123456789ABC
```

The MMU must translate this into a physical address.

Before walking tables, the MMU will:

1. check the TLB
2. if no hit, perform a page-table walk

For this example, assume there is a **TLB miss**.

---

## 3. Translation Starts from TTBR

Assume the address range selects **TTBR0_EL1**.

Suppose:

```text
TTBR0_EL1 = 0x0000000081000000
```

This means:

- the Level 0 translation table starts at physical address `0x81000000`

So the MMU begins the walk there.

---

## 4. Address Breakdown Idea

For a 4 KB granule, the lower 12 bits are the page offset.

### Offset

From:

```text
VA = 0x0000123456789ABC
```

The lower 12 bits are:

```text
offset = 0xABC
```

That offset will remain unchanged throughout translation.

The remaining upper bits are used as indexes into the translation tables.

Conceptually:

```text
VA = [L0 index][L1 index][L2 index][L3 index][0xABC]
```

We do not need to compute each exact index bit-by-bit for the teaching idea. What matters is:

- each level chooses one entry
- the chosen entry either points to the next table or gives a final block/page mapping

---

## 5. Level 0 Walk

Base of Level 0 table:

```text
L0 table base = 0x81000000
```

The MMU uses the **L0 index bits** from the virtual address to pick one entry.

Assume the selected Level 0 entry says:

```text
L0 entry -> next table at 0x82000000
```

This is a **table descriptor**.

Meaning:

- translation is not finished
- continue at Level 1 table located at `0x82000000`

---

## 6. Level 1 Walk

Base of Level 1 table:

```text
L1 table base = 0x82000000
```

The MMU uses the **L1 index bits** to select one entry.

Assume the selected entry says:

```text
L1 entry -> next table at 0x83000000
```

Again, this is a **table descriptor**.

Meaning:

- continue to Level 2 table

---

## 7. Level 2 Walk

Base of Level 2 table:

```text
L2 table base = 0x83000000
```

The MMU uses the **L2 index bits** and finds an entry.

Assume the selected entry says:

```text
L2 entry -> next table at 0x84000000
```

Again, translation continues.

---

## 8. Level 3 Walk

Base of Level 3 table:

```text
L3 table base = 0x84000000
```

The MMU uses the **L3 index bits** and selects the final entry.

Assume this Level 3 entry is a **page descriptor** and says:

```text
physical page base = 0x45603000
```

This means the virtual address belongs to the page whose physical base is `0x45603000`.

---

## 9. Forming the Final Physical Address

Now the MMU combines:

- physical page base = `0x45603000`
- page offset = `0xABC`

So:

```text
PA = 0x45603000 + 0xABC
PA = 0x45603ABC
```

### Final result

```text
Virtual Address  = 0x0000123456789ABC
Physical Address = 0x45603ABC
```

---

## 10. Full Walk Summary

The whole walk looks like this:

```text
VA = 0x0000123456789ABC

TTBR0_EL1 -> 0x81000000   (Level 0 table base)
L0 entry   -> 0x82000000   (Level 1 table)
L1 entry   -> 0x83000000   (Level 2 table)
L2 entry   -> 0x84000000   (Level 3 table)
L3 entry   -> 0x45603000   (physical page base)
offset     -> 0xABC

Final PA   -> 0x45603ABC
```

---

## 11. If a Block Descriptor Appears Earlier

The walk does not always need to go to Level 3.

Suppose the Level 2 entry is a **block descriptor** instead of a table descriptor.

Then translation could stop early.

Example:

```text
L2 entry -> block base address = 0x70000000
```

Now the final physical address would be formed using:

- block base address
- lower address bits that act as offset within the block

### Meaning

This reduces table-walk depth and is useful for large continuous regions.

But it gives less fine-grained control than page mapping.

---

## 12. What If the Entry Is Invalid?

If any selected entry is invalid, translation fails.

For example:

- invalid L1 entry
- invalid L2 entry
- permission violation at final descriptor

Then the MMU raises a fault such as:

- translation fault
- access fault
- permission fault

The operating system then handles the exception.

---

## 13. Role of the TLB in This Example

In the example above, we assumed a **TLB miss**.

After the MMU completes the walk and gets:

```text
VA page -> PA page
```

it usually stores that mapping in the TLB.

So next time the CPU accesses the same page:

- translation can happen immediately
- no full table walk is needed

This is why the TLB is important for performance.

---

## 14. Why the Offset Does Not Change

This is one of the most important points.

The offset stays the same because translation maps a **page to a page**.

The byte position inside the page does not change.

### Example again

If:

- virtual address ends with `0xABC`
- page size is 4 KB

then `0xABC` is simply the byte offset inside the page.

After translation, the physical page changes, but that byte position remains `0xABC`.

---

## 15. Short Intuition

You can think of the walk like this:

- Level 0 says which big region to search
- Level 1 narrows it further
- Level 2 narrows it further
- Level 3 gives the final page
- offset gives the exact byte inside that page

So the translation process is:

> choose table -> choose subtable -> choose subtable -> choose page -> add offset

---

## 16. Final Short Summary

### Example result

```text
VA = 0x0000123456789ABC
PA = 0x45603ABC
```

### Steps

1. select TTBR
2. use L0 index
3. use L1 index
4. use L2 index
5. use L3 index
6. get physical page base
7. add unchanged offset

### Key lesson

**The MMU translates the page base through descriptors, while the page offset is copied unchanged into the final physical address.**
