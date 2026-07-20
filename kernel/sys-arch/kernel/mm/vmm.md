# Virtual Memory Manager (VMM)

This document describes the virtual memory management subsystem in the Kyronix kernel.

## Overview

The VMM manages x86-64 4-level page tables (PML4 -> PDPT -> PD -> PT) with 4 KiB page granularity and NX bit support.

## Initialization

`vmm_init()` enables the NX bit via `IA32_EFER` MSR bit 11, then reads the current CR3 to initialize the kernel address space.

## Address Spaces

Each process has its own `vmm_space_t` containing:

- `pml4_phys` -- physical address of the PML4
- `vmas[2048]` -- fixed array of VMA descriptors

New address spaces share the kernel half (PML4 entries 256-511) with the kernel space. User-half PML4 entries start zeroed.

## Operations

| Function | Description |
|----------|-------------|
| `vmm_map(sp, virt, phys, flags)` | Map a virtual page to a physical page |
| `vmm_unmap(sp, virt)` | Remove a mapping |
| `vmm_virt_to_phys(sp, virt)` | Translate virtual to physical address |
| `vmm_protect(sp, virt, flags)` | Change page protection flags |
| `vmm_space_new()` | Create a new address space |
| `vmm_space_free(sp)` | Free an address space (user half only) |
| `vmm_switch(sp)` | Switch to a different address space (load CR3) |
| `vmm_fork_user(dst, src)` | Copy user-half page tables for fork() |

## Page Flags

| Flag | Description |
|------|-------------|
| `VMM_PRESENT` | Page is present |
| `VMM_WRITE` | Page is writable |
| `VMM_USER` | Page is accessible from ring 3 |
| `VMM_NX` | No-execute bit |
| `VMM_PCD` | Page cache disable (uncacheable) |

Predefined flag combinations:

- `VMM_KCODE` = PRESENT (kernel code)
- `VMM_KDATA` = PRESENT | WRITE | NX (kernel data)
- `VMM_UCODE` = PRESENT | USER (user code)
- `VMM_UDATA` = PRESENT | WRITE | USER | NX (user data)

## Demand Paging

`vmm_user_range_fault_in()` implements demand paging:

1. Checks if each page in the range is present and accessible
2. For missing pages, checks if a VMA covers the address
3. If yes, allocates a zeroed physical page and maps it with the VMA's flags
4. This enables lazy allocation for `mmap(MAP_ANON)` regions

## User Range Validation

- `vmm_user_range_ok()` -- verifies every page is mapped and accessible
- `vmm_user_range_fault_in()` -- verifies + faults in missing pages

## Fork

`vmm_fork_user()` copies the entire user-half page table hierarchy:

1. Copies VMA metadata via `vma_copy()`
2. Walks the source PML4 entries 0-255
3. For each present page, allocates a new physical page, copies the content, and maps it in the destination

> **Note:** This is a full-copy fork (not copy-on-write). All user pages are duplicated immediately.

## Pool

The VMM maintains a pool of 256 address spaces (`VMM_MAX_SPACES`). `vmm_space_new()` allocates from this pool.
