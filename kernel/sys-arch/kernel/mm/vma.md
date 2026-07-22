# VMA

This document describes the Virtual Memory Area (VMA) subsystem in the Kyronix kernel. It is the child of [Memory Management](sys-arch/kernel/mm/index.md).

## Source

`kernel/mm/vma.c`, `kernel/mm/vma.h`

## Overview

VMAs track which virtual address ranges are mapped in each address space. They are used for demand paging, mmap/munmap tracking, and page fault handling.

## Protection Constants

| Constant | Value | Description |
|---|---|---|
| `PROT_READ` | 0x1 | Read permission |
| `PROT_WRITE` | 0x2 | Write permission |
| `PROT_EXEC` | 0x4 | Execute permission |

## Data Structure

```c
typedef struct {
    uint64_t start;         // Start virtual address
    uint64_t end;           // End virtual address
    uint32_t prot;          // Protection flags (PROT_READ/WRITE/EXEC)
    uint32_t map_flags;     // VMM mapping flags
    uint8_t  used;          // Slot in use
    uint8_t  free_on_unmap; // Free physical pages on unmap
} vmm_vma_t;
```

VMAs are stored in a flat array of 2048 entries (`VMM_VMA_MAX`) inside each `vmm_space_t`. All lookups are linear scans.

## Functions

| Function | Description |
|---|---|
| `vma_reset(sp)` | Zero all VMA slots |
| `vma_copy(dst, src)` | Copy VMA array between address spaces |
| `vma_conflicts(sp, start, len)` | Check for overlapping VMAs |
| `vma_add(sp, start, len, prot, flags, free_on_unmap)` | Register a new VMA |
| `vma_remove(sp, start, len)` | Remove a VMA range (with splitting) |
| `vma_remove_overlaps(sp, start, len)` | Remove all overlapping VMAs |
| `vma_protect(sp, start, len, prot)` | Change protection on a range |
| `vma_page_fault_allowed(sp, addr, write, exec)` | Check if a page fault can be handled |
| `vma_page_flags(sp, addr)` | Convert VMA protection to PTE flags |
| `vma_range_ok(sp, start, len)` | Validate range is fully covered by VMAs |

## VMA Splitting

When removing a sub-range from a VMA, four cases apply:

1. **No leftovers**: Clear the VMA entirely
2. **Left only**: Shrink `v->end = start`
3. **Right only**: Shrink `v->start = end`
4. **Split**: Create two VMAs (left and right of the hole)

Case 4 requires an empty VMA slot and may fail with `-ENOMEM`.

## Page Fault Integration

`vma_page_fault_allowed()` is called by the VMM's demand paging path. It returns true only if:

- A VMA covers the faulting address
- The VMA has `free_on_unmap` set
- The VMA has sufficient permissions (PROT_READ minimum, PROT_WRITE for writes, PROT_EXEC for instruction fetches)

This enables lazy allocation: pages are allocated on first access rather than at mmap time.

Last reviewed: 2026-07-22
