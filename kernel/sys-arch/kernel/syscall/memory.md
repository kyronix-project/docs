# Memory Management Syscalls

This document describes the memory management system calls (syscalls) in the Kyronix kernel. It is the child of [Syscalls](index.md).

## Syscall Table

- 9 `mmap` - Map memory or files into address space
- 10 `mprotect` - Set memory protection on a region
- 11 `munmap` - Unmap memory from address space
- 12 `brk` - Change data segment size

- 25 `mremap` - Remap a virtual memory area (VMA)
- 26 `msync` - Synchronize memory with storage (noop for ramfs)
- 27 `mincore` - Determine page residency in memory
- 28 `madvise` - Provide advice about memory usage (noop)

- 29 `shmget` - Get System V shared memory segment
- 30 `shmat` - Attach shared memory to address space
- 31 `shmctl` - Control shared memory operations
- 67 `shmdt` - Detach shared memory from address space

## mmap Flags

The `mmap` syscall accepts the following flags:

- `MAP_ANON` - Create an anonymous mapping (not backed by a file)
- `MAP_FIXED` - Place the mapping at the exact address specified
- `MAP_PRIVATE` - Create a private copy-on-write (COW) mapping
- `MAP_SHARED` - Create a shared mapping visible to other processes

## mmap Details

The `mmap` syscall uses `mmap_pick_addr()` to select a virtual address for anonymous mappings. The address selection uses a bump allocator that starts at:

```
p->mmap_bump = 0x0000500000000000 + random_offset
```

The `mmap` implementation supports:

- Anonymous mappings (file descriptor is -1)
- File-backed mappings (valid file descriptor)
- Character device custom mappings (User I/O (UIO))

## brk Details

The `brk` syscall implements the classic break mechanism for heap management:

- Operates with page-granularity (minimum allocation is one page)
- Allocates zeroed pages via `pmm_alloc_zeroed()`
- Tracks `pages_alloc` and `pages_freed` counters
- Expands the data segment by mapping new pages
- Contracts the data segment by unmapping pages

## mprotect Details

The `mprotect` syscall changes both:

- Virtual Memory Area (VMA) metadata (protection flags)
- Page table entries (hardware-level permissions)

## Shared Memory (SHM)

The SHM syscalls implement System V shared memory with the following characteristics:

- Requires `JAILF_IPC` flag for jail isolation
- Maximum of 64 shared memory segments per process
- Maximum of 4096 pages (16 MiB at 4 KiB pages) per segment
- Supports attach, detach, and control operations

Last reviewed: 2026-07-22
