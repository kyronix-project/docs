# Memory Management

This document describes the memory management subsystem of the Kyronix kernel. It is the child of [Kernel](sys-arch/kernel/index.md) and parent of memory management component documents.

## Architecture

The memory management subsystem consists of four layers:

- **LL-Free** (`llfree.c`) -- Lock-free physical frame allocator
- **PMM** (`pmm.c`) -- Physical page allocation
- **VMM / VMA** (`vmm.c`, `vma.c`) -- Virtual memory and area tracking
- **Heap / SHM** (`heap.c`, `shm.c`) -- Kernel heap and shared memory

## Components

| Component | Source | Description |
|---|---|---|
| PMM | `mm/pmm.c` | Physical page allocation via LL-Free |
| VMM | `mm/vmm.c` | 4-level page table management |
| VMA | `mm/vma.c` | Virtual Memory Area tracking |
| Heap | `mm/heap.c` | Kernel heap (first-fit linked list) |
| SHM | `mm/shm.c` | SysV shared memory (up to 64 segments) |
| LL-Free | `mm/llfree.c` | Lock-free physical frame allocator |
| KmemLeak | `mm/kmemleak.c` | Kernel memory leak detector |

## Key Constants

| Constant | Value | Description |
|---|---|---|
| `PAGE_SIZE` | 4096 | Page frame size |
| `PAGE_SHIFT` | 12 | Bit shift for page-to-byte conversion |
| `HEAP_START` | `0xffff910000000000` | Heap virtual address base |
| `HEAP_MAX` | `0xffff920000000000` | Maximum heap address (4 GiB) |
| `USER_LIMIT` | `0x800000000000` | Top of user half (128 TiB) |
| `VMM_MAX_SPACES` | 256 | Maximum concurrent address spaces |
| `VMM_VMA_MAX` | 2048 | Maximum VMAs per address space |

Last reviewed: 2026-07-22
