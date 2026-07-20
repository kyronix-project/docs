# Memory Management

This section describes the memory management subsystem of the Kyronix kernel.

## Components

- [Physical Memory Manager (PMM)](pmm.md) -- bitmap-based physical page allocator
- [Virtual Memory Manager (VMM)](vmm.md) -- 4-level page table management
- [Heap Allocator](heap.md) -- kernel heap (kmalloc/kfree)
- [Virtual Memory Areas (VMA)](vma.md) -- user-space memory region tracking

## Address Space Layout

```
0x0000000000000000 +-----------------------+
                   |     User Space        |
                   |   (up to 128 TiB)     |
0x0000800000000000 +-----------------------+ USER_LIMIT
                   |                       |
0xffff800000000000 +-----------------------+ HHDM (physical memory direct map)
                   |    Kernel Space       |
0xffffffff80000000 +-----------------------+ Kernel .text/.data/.bss
                   |    Kernel Heap        |
0xffff910000000000 +-----------------------+ HEAP_START
0xffff920000000000 +-----------------------+ HEAP_MAX
                   |   Kernel Stacks       |
0xffff920000000000 +-----------------------+ KSTACK_VA_BASE
```

## Key Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `PAGE_SIZE` | 4096 | Physical page size |
| `USER_LIMIT` | 0x800000000000 | User/kernel address boundary |
| `HEAP_START` | 0xffff910000000000 | Kernel heap base |
| `HEAP_MAX` | 0xffff920000000000 | Kernel heap ceiling |
| `KSTACK_PAGES` | 16 | Kernel stack pages per process |
| `VMM_VMA_MAX` | 2048 | Max VMAs per address space |
