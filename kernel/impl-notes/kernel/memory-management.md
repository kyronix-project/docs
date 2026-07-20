# Memory Management Implementation

This document describes the implementation details of the Kyronix memory management subsystem.

## Physical Memory Manager (PMM)

### Bitmap Layout

The PMM uses a flat bitmap where each bit represents one 4 KiB page:

- Bit = 1: page is free
- Bit = 0: page is used

The bitmap is stored in a physically contiguous region of memory, accessed via the HHDM offset.

### Allocation Algorithm

`pmm_alloc()` scans the bitmap word by word:

1. Skip words that are all-zero (no free pages)
2. Use `__builtin_ctzll` to find the first set bit (free page)
3. Clear the bit (mark as used)
4. Return the physical address

This gives O(n/64) worst-case performance.

### Initialization

The bitmap is allocated from the first usable Limine memory map entry large enough to hold it. The bitmap pages are then marked as used.

## Virtual Memory Manager (VMM)

### Page Table Walk

`vmm_map()` performs a 4-level page table walk (PML4 -> PDPT -> PD -> PT). Intermediate entries are always PRESENT|WRITE|USER. Leaf PTEs restrict access via the actual flags.

### Address Space Pool

A fixed pool of 256 address spaces. Each new space copies kernel PML4 entries (256-511) from the kernel space.

### Fork

`vmm_fork_user()` walks the source user-half page tables and copies every page immediately (full-copy fork, not COW).

## Heap Allocator

### Block Layout

Each allocation has a 32-byte header (size, free flag, prev/next pointers). Allocations are 16-byte aligned.

### Coalescing

On `kfree()`, adjacent free blocks are merged with both the previous and next blocks.

### Growth

When no free block fits, the heap grows by at least 16 pages (64 KiB). New pages are allocated from PMM and mapped into kernel space.

## VMA System

### Split Operation

The core operation is `split_for_hole()`: when removing a range from the middle of a VMA, the VMA is split into up to two pieces. Left-only and right-only truncations do not require an extra slot.

### Protect Operation

`vma_protect()` splits VMAs at range boundaries, then re-adds them with the new protection flags.
