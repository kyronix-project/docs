# Heap

This document describes the kernel heap allocator in the Kyronix kernel. It is the child of [Memory Management](sys-arch/kernel/mm/index.md).

## Source

`kernel/mm/heap.c`, `kernel/mm/heap.h`

## Overview

The kernel heap provides dynamic memory allocation via `kmalloc()` and `kfree()`. It uses a first-fit linked-list allocator with block coalescing, backed by physically contiguous pages from the PMM.

## Address Range

| Constant | Value | Description |
|---|---|---|
| `HEAP_START` | `0xffff910000000000` | Heap virtual base |
| `HEAP_MAX` | `0xffff920000000000` | Maximum heap (4 GiB) |

## Data Structure

```c
typedef struct block_hdr {
    uint64_t         size;  // Payload size (excluding header)
    uint64_t         free;  // 1 = free, 0 = allocated
    struct block_hdr *prev;
    struct block_hdr *next;
} block_hdr_t;  // 32 bytes
```

## Allocation Algorithm

### `kmalloc(size)`

1. Align requested size to 16 bytes
2. Disable IRQs, acquire spinlock
3. Walk linked list for first-fit free block
4. If no block found, call `heap_grow()` (16 pages = 64 KiB minimum)
5. If block is large enough to split (remaining >= 48 bytes), split it
6. Mark block as allocated, update stats

### `kfree(ptr)`

1. Disable IRQs, acquire spinlock
2. Mark block free
3. Forward coalesce: merge with next block if free
4. Backward coalesce: merge with previous block if free

### `heap_grow(min_payload)`

1. Compute pages needed: `ceil(min_payload / PAGE_SIZE)` (minimum 16 pages)
2. Allocate contiguous pages via `pmm_alloc_contiguous()`
3. Map each page into heap range with `VMM_KDATA` flags
4. Append new free block to linked list

## Functions

| Function | Description |
|---|---|
| `heap_init()` | Initial heap growth (64 KiB) |
| `kmalloc(size)` | First-fit allocation with 16-byte alignment |
| `kcalloc(nmemb, size)` | Allocation + zero-fill |
| `krealloc(ptr, new_size)` | Resize (copy if needed) |
| `kfree(ptr)` | Free with coalescing |
| `heap_stats()` | Print block count, used/free in KiB |
| `heap_alloc_delta()` | Net allocated bytes (alloc - free) |
| `heap_walk_used(callback, user)` | Walk all allocated blocks (for kmemleak) |

## Thread Safety

The heap uses a spinlock with IRQ save/restore for all operations. This ensures safe concurrent access from multiple CPUs and interrupt handlers.

Last reviewed: 2026-07-22
