# Physical Memory Manager (PMM)

This document describes the physical page allocator in the Kyronix kernel.

## Overview

The PMM manages physical memory using a bitmap-based allocator. One bit per 4 KiB page: 1 = free, 0 = used.

## Initialization

`pmm_init()` is called during boot with the Limine memory map and HHDM offset:

1. Scans the Limine memory map to find the highest usable physical address
2. Allocates a bitmap from the first sufficiently large usable region
3. Marks all usable pages as free (bit = 1)
4. Marks the bitmap pages themselves as used (bit = 0)

## Allocation

| Function | Description |
|----------|-------------|
| `pmm_alloc()` | Allocate a single 4 KiB page |
| `pmm_alloc_zeroed()` | Allocate and zero-fill a page |
| `pmm_alloc_contiguous(n)` | Allocate n physically-consecutive pages |
| `pmm_free(phys)` | Free a previously allocated page |

`pmm_alloc()` uses `__builtin_ctzll` (count trailing zeros) to find the first free bit in the bitmap, giving O(n/64) worst-case performance.

`pmm_alloc_contiguous()` performs a brute-force linear scan for N consecutive free pages.

## Statistics

| Function | Description |
|----------|-------------|
| `pmm_free_pages()` | Number of currently free pages |
| `pmm_total_pages()` | Total pages in the system |
| `pmm_usable_pages()` | Total usable pages |
| `pmm_alloc_total()` | Lifetime total pages allocated |
| `pmm_free_total()` | Lifetime total pages freed |

## Address Conversion

| Function | Description |
|----------|-------------|
| `phys_to_virt(phys)` | Convert physical address to virtual (HHDM offset) |
| `virt_to_phys(virt)` | Convert virtual address to physical |

These are O(1) offset-based conversions using the HHDM offset provided by Limine.

## Integration

The PMM integrates with `CONFIG_KMEMLEAK` (if enabled) to track all page allocations for leak detection via `kmemleak_track_page` / `kmemleak_untrack_page`.
