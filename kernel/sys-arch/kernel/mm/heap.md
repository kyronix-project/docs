# Heap Allocator

This document describes the kernel heap allocator in the Kyronix kernel.

## Overview

The heap allocator provides `kmalloc`/`kfree`-style dynamic memory allocation for kernel objects. It uses a first-fit linked-list design in the kernel virtual address space.

## Address Range

- Start: `0xffff910000000000` (`HEAP_START`)
- End: `0xffff920000000000` (`HEAP_MAX`)

## Block Structure

Each allocation is preceded by a 32-byte block header:

```c
typedef struct block_hdr {
    uint64_t size;     // payload size (excluding header)
    uint64_t free;     // 1 = free, 0 = allocated
    struct block_hdr *prev;
    struct block_hdr *next;
} block_hdr_t;
```

## Operations

| Function | Description |
|----------|-------------|
| `kmalloc(size)` | Allocate `size` bytes (16-byte aligned) |
| `kcalloc(nmemb, size)` | Allocate zeroed memory |
| `krealloc(ptr, new_size)` | Reallocate to a new size |
| `kfree(ptr)` | Free a previous allocation |

## Allocation Strategy

1. Scan the linked list for the first free block that fits
2. If no block fits, grow the heap by at least 16 pages (64 KiB)
3. If the block is large enough, split it into two blocks
4. Mark the block as allocated

## Deallocation

1. Mark the block as free
2. Coalesce with the next free block if adjacent
3. Coalesce with the previous free block if adjacent

## Heap Growth

When the heap needs more space:

1. Calculate the number of pages needed (at least 16 pages)
2. For each page, allocate a physical page via `pmm_alloc()`
3. Map it into the kernel address space via `vmm_map()`
4. Create a new free block at the break address

## Thread Safety

All heap operations are protected by:

1. IRQ save/restore (`irq_save` / `irq_restore`)
2. Spinlock (`g_heap_lock`)

This ensures safe access from interrupt context.

## Diagnostics

| Function | Description |
|----------|-------------|
| `heap_stats()` | Log block count, used/free bytes |
| `heap_alloc_delta()` | Net allocated bytes (allocated - freed) |
| `heap_brk()` | Current heap break address |
| `heap_walk_used(callback)` | Iterate over all allocated blocks |

The `heap_walk_used` function is used by the `kmemleak` subsystem for leak scanning.
