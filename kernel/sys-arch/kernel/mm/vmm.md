# VMM

This document describes the Virtual Memory Manager (VMM) in the Kyronix kernel. It is the child of [Memory Management](sys-arch/kernel/mm/index.md).

## Source

`kernel/mm/vmm.c`, `kernel/mm/vmm.h`

## Overview

The VMM manages 4-level x86-64 page tables (PML4 -> PDPT -> PD -> PT) for both kernel and user address spaces. It provides page mapping, unmapping, protection changes, and demand paging support.

## Address Space Layout

- **User half**: `0x000000000000` to `0x800000000000` (128 TiB)
- **Kernel half**: `0x800000000000` to `0xFFFFFFFFFFFF` (128 TiB)

Kernel page table entries (PML4 indices 256-511) are shared across all address spaces.

## Composite Flags

| Name | Value | Meaning |
|---|---|---|
| `VMM_KCODE` | `PRESENT` | Kernel code: present, NX off |
| `VMM_KDATA` | `PRESENT \| WRITE \| NX` | Kernel data: present, writable, NX |
| `VMM_UCODE` | `PRESENT \| USER` | User code: present, user, NX off |
| `VMM_UDATA` | `PRESENT \| WRITE \| USER \| NX` | User data: present, writable, user, NX |

## Functions

| Function | Description |
|---|---|
| `vmm_init()` | Enable NX bit via EFER, read kernel PML4 from CR3 |
| `vmm_map(sp, virt, phys, flags)` | Map a single page (allocates intermediate tables as needed) |
| `vmm_unmap(sp, virt)` | Unmap a single page (zeroes leaf PTE) |
| `vmm_protect(sp, virt, flags)` | Change flags on existing mapping |
| `vmm_virt_to_phys(sp, virt)` | Walk page tables for virtual-to-physical translation |
| `vmm_user_range_ok(sp, virt, len, write)` | Validate user range accessibility |
| `vmm_user_range_fault_in(sp, virt, len, write)` | Demand page user range (allocates on fault) |
| `vmm_space_new()` | Create new address space (copies kernel half) |
| `vmm_space_free(sp)` | Free all user-half page tables and frames |
| `vmm_switch(sp)` | Switch address space (write CR3) |
| `vmm_fork_user(dst, src)` | Deep-copy entire user-half address space |

## Page Table Indexing

| Level | Index Macro | Bits |
|---|---|---|
| PML4 | `(va >> 39) & 0x1FF` | Bits 39-47 |
| PDPT | `(va >> 30) & 0x1FF` | Bits 30-38 |
| PD | `(va >> 21) & 0x1FF` | Bits 21-29 |
| PT | `(va >> 12) & 0x1FF` | Bits 12-20 |

## Demand Paging

`vmm_user_range_fault_in()` implements demand paging:

1. Check if the VMA subsystem permits the fault (`vma_page_fault_allowed`)
2. Allocate a zeroed physical page via `pmm_alloc_zeroed()`
3. Map it into the address space with appropriate flags
4. This allows lazy allocation of user memory on first access

## Address Space Management

- Maximum 256 concurrent address spaces (`VMM_MAX_SPACES`)
- Each space stores its PML4 physical address and a flat array of up to 2048 VMAs
- `vmm_space_new()` copies kernel half entries from `g_kernel_space`
- `vmm_space_free()` recursively frees all user-half page table levels

Last reviewed: 2026-07-22
