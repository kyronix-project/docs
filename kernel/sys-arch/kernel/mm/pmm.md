# PMM

This document describes the Physical Memory Manager (PMM) in the Kyronix kernel. It is the child of [Memory Management](sys-arch/kernel/mm/index.md).

## Source

`kernel/mm/pmm.c`, `kernel/mm/pmm.h`

## Overview

The PMM manages physical page frames using the LL-Free lock-free allocator. It provides allocation and deallocation of 4 KiB page frames, with support for contiguous multi-page allocations.

## Key Constants

| Constant | Value | Description |
|---|---|---|
| `PAGE_SIZE` | 4096 | Size of one page frame |
| `PAGE_SHIFT` | 12 | Bits to shift for page count |
| `ZPOOL_SIZE` | 32 | Pre-allocated zeroed page pool size |

## Macros

| Macro | Description |
|---|---|
| `PAGE_ALIGN_UP(x)` | Round up to next page boundary |
| `PAGE_ALIGN_DOWN(x)` | Round down to page boundary |
| `phys_to_virt(phys)` | Physical to virtual via HHDM offset |
| `virt_to_phys(virt)` | Virtual to physical via HHDM offset |

## Functions

| Function | Description |
|---|---|
| `pmm_init(memmap, hhdm_offset, kernel_end)` | Initialize from Limine memory map |
| `pmm_alloc()` | Allocate one frame (order 0) |
| `pmm_alloc_zeroed()` | Allocate one zeroed frame (pool or alloc+memset) |
| `pmm_alloc_contiguous(n)` | Allocate n physically contiguous pages |
| `pmm_free(phys)` | Return a frame to the allocator |
| `pmm_free_pages()` | Return count of free frames |
| `pmm_total_pages()` | Return total managed frames |
| `pmm_usable_pages()` | Return frames marked usable by Limine |

## Initialization

1. Parse Limine memory map to find highest usable address
2. Compute total frames from usable regions
3. Allocate LL-Free metadata from the largest usable region (after `kernel_end_phys`)
4. Initialize LL-Free with `LLFREE_INIT_FREE` (all frames free)
5. Fill the zero-page pool (32 pre-allocated zeroed pages)

Requires at least 512 managed frames.

## Zero-Page Pool

The PMM maintains a stack of 32 pre-allocated zeroed pages (`g_zpool`). `pmm_alloc_zeroed()` first checks this pool; if empty, falls back to `pmm_alloc()` + `memset(0)`. This avoids repeated zeroing for common allocations.

## LL-Free Integration

The PMM delegates to the LL-Free allocator for all frame management. LL-Free is a three-tier lock-free buddy-like allocator with CPU-local reservations for scalability:

- **Frame**: 4 KiB base unit
- **Child**: 512 frames = 2 MiB (huge page boundary)
- **Tree**: 8 children = 4096 frames = 16 MiB
- **Local**: Per-CPU reservation of one tree

Maximum allocatable order is 12 (4096 frames = 16 MiB contiguous).

Last reviewed: 2026-07-22
