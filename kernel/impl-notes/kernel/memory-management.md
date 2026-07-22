# Memory Management

This document describes the memory management implementation details in the Kyronix kernel. It is the child of [Kernel Implementation Notes](index.md).

The Kyronix kernel implements a layered memory management architecture comprising a Physical Memory Manager (PMM), Virtual Memory Manager (VMM), kernel heap, Shared Memory (SHM) subsystem, and a runtime memory leak detector.

## Physical Memory Manager (PMM)

### Initialization

1. Parse the Limine boot protocol memory map to identify usable physical memory regions.
2. Find the highest usable physical address across all regions.
3. Compute the total number of physical frames from the highest usable address.
4. Allocate LL-Free metadata from the largest contiguous usable region.

### LL-Free Three-Tier Architecture

The LL-Free allocator organizes physical frames into a three-tier hierarchy:

1. **Frame:** A single 4 KiB physical page.
2. **Child:** A group of 512 frames, totaling 2 MiB.
3. **Tree:** A group of 8 children, totaling 4096 frames (16 MiB).

### CPU-Local Reservations

Each CPU reserves one tree from the LL-Free allocator for lock-free allocation, eliminating cross-CPU contention on the fast path.

### Zero-Page Pool

A pool of 32 pre-allocated zeroed physical pages provides fast allocation for `pmm_alloc_zeroed` without requiring a memset on each allocation.

## Virtual Memory Manager (VMM)

### Page Table Management

The VMM performs a 4-level page table walk (PML4 → PDPT → PD → PT) to resolve and manipulate virtual-to-physical address mappings.

### Demand Paging

Unmapped pages within valid Virtual Memory Area (VMA) regions trigger demand paging via `vmm_user_range_fault_in`, which allocates physical frames and installs page table entries on demand.

## Virtual Memory Areas (VMA)

- VMAs are stored in a flat array of 2048 entries per process.
- Lookups use a linear scan of the array.
- `split_for_hole` splits an existing VMA into two smaller VMAs when a region must be removed from the middle of a mapping.

## Kernel Heap

- The heap uses a first-fit allocator with a doubly-linked free list.
- Coalescing is performed in both the forward and backward directions on free.
- All allocations are aligned to 16-byte boundaries.
- The heap grows in 64 KiB increments from the initial allocation.

## Shared Memory (SHM)

- Shared Memory follows the System V Inter-Process Communication (IPC) model.
- A maximum of 64 SHM segments are supported.
- Jail isolation is enforced via the `JAILF_IPC` flag, which prevents processes within a jail from accessing SHM segments outside the jail.

## Memory Leak Detector (kmemleak)

The kmemleak subsystem implements a mark-and-sweep leak detector that scans the following memory regions for allocated pointers:

1. `.data` section — Initialized global and static variables.
2. `.bss` section — Uninitialized global and static variables.
3. Process table — All live process structures.
4. Heap — All live heap allocations.
5. Page tables — All mapped page table entries.

Last reviewed: 2026-07-22
