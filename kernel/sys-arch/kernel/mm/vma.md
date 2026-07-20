# Virtual Memory Areas (VMA)

This document describes the VMA (Virtual Memory Area) tracking system in the Kyronix kernel.

## Overview

VMAs track user-space memory regions, recording their address range, protection flags, and ownership. They support demand paging, memory protection, and proper cleanup on unmap.

## VMA Structure

```c
typedef struct {
    uint64_t start;       // start address (inclusive)
    uint64_t end;         // end address (exclusive)
    uint32_t prot;        // protection flags (PROT_READ, PROT_WRITE, PROT_EXEC)
    uint32_t map_flags;   // mmap flags (MAP_ANON, MAP_FIXED, etc.)
    uint8_t used;         // 1 = slot in use
    uint8_t free_on_unmap; // 1 = pages are owned, free on munmap
} vmm_vma_t;
```

Each address space contains a flat array of 2048 VMA entries.

## Protection Flags

| Flag | Value | Description |
|------|-------|-------------|
| `PROT_READ` | 0x1 | Page is readable |
| `PROT_WRITE` | 0x2 | Page is writable |
| `PROT_EXEC` | 0x4 | Page is executable |

## Operations

| Function | Description |
|----------|-------------|
| `vma_add(sp, start, len, prot, map_flags, free_on_unmap)` | Add a new VMA |
| `vma_remove(sp, start, len)` | Remove VMAs covering a range |
| `vma_remove_overlaps(sp, start, len)` | Remove overlapping portions of VMAs |
| `vma_protect(sp, start, len, prot)` | Change protection on a range |
| `vma_conflicts(sp, start, len)` | Check if a range overlaps existing VMAs |
| `vma_range_ok(sp, start, len)` | Check if entire range is covered by VMAs |
| `vma_copy(dst, src)` | Copy VMA array (for fork) |
| `vma_reset(sp)` | Clear all VMAs (for new address space) |

## Demand Paging Integration

| Function | Description |
|----------|-------------|
| `vma_page_fault_allowed(sp, addr, write, exec)` | Check if a page fault should be handled |
| `vma_page_flags(sp, addr)` | Get page table flags for a VMA region |
| `vma_page_mapped(sp, addr)` | Check if an address has a VMA |
| `vma_page_owned(sp, addr)` | Check if pages in this region are owned |

When a page fault occurs, the VMM calls `vma_page_fault_allowed()` to determine if the fault is in a demand-paggable region. If the VMA has `free_on_unmap` set, the page is allocated and mapped on demand.

## Split and Merge

The `split_for_hole()` function splits a VMA when a range is removed from the middle:

- If the removed range is in the left portion, the VMA is truncated
- If the removed range is in the right portion, the VMA's start is moved
- If the removed range is in the middle, the VMA is split into two (requires a free slot)

`vma_protect()` uses split + re-add to change protection on sub-ranges.
