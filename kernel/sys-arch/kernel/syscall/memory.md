# Memory Management Syscalls

This document describes the memory-related syscalls in the Kyronix kernel.

## Syscalls

| Syscall | Number | Description |
|---------|--------|-------------|
| `mmap` | 9 | Map files or anonymous memory into address space |
| `mprotect` | 10 | Change memory protection |
| `munmap` | 11 | Unmap memory region |
| `brk` | 12 | Change data segment size |
| `mremap` | 25 | Remap a memory region |
| `madvise` | 28 | Give advice about memory usage |

## mmap

`mmap(addr, length, prot, flags, fd, offset)` maps memory into the process address space.

### Flags

| Flag | Description |
|------|-------------|
| `MAP_SHARED` | Shared mapping (changes visible to other processes) |
| `MAP_PRIVATE` | Private mapping (copy-on-write) |
| `MAP_ANONYMOUS` | Anonymous memory (no file backing) |
| `MAP_FIXED` | Replace existing mapping at addr |
| `MAP_FIXED_NOREPLACE` | Map at addr without replacing |

### Mapping Types

1. **Anonymous (`MAP_ANON`):** Fresh zeroed pages from PMM
2. **File-backed private (`MAP_PRIVATE`):** Copies file data into fresh physical pages
3. **Device memory:** Custom mmap for UIO physical BAR mapping

### VMA Tracking

Each mmap creates a VMA entry tracking the address range, protection, and ownership. VMAs are used for demand paging and proper cleanup on munmap.

## munmap

`munmap(addr, length)` removes a memory mapping:

1. Remove overlapping VMAs via `vma_remove()`
2. For pages with `free_on_unmap`, unmap and free the physical pages
3. Flush TLB with `invlpg`

## mprotect

`mprotect(addr, length, prot)` changes memory protection:

1. Validate the range covers existing VMAs
2. Split VMAs at range boundaries
3. Re-add VMAs with the new protection flags
4. Update page table entries via `vmm_protect()`

## brk

`brk(addr)` adjusts the program break (heap end):

- If `addr > current brk`: allocates new pages via `pmm_alloc()` and maps them
- If `addr < current brk`: frees pages and unmaps them
- Returns the new break address

## mremap

`mremap(old_addr, old_size, new_size, flags)` remaps a memory region:

- `MREMAP_MAYMOVE`: allows moving the region
- `MREMAP_FIXED`: maps to a specific address

Copies old pages to the new location if moved.

## Memory Accounting

Each process tracks:

- `pages_alloc` -- pages allocated via brk/mmap
- `pages_freed` -- pages freed via munmap/shrink brk

These are used for memory usage reporting.
