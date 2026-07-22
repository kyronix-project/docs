# Limine Protocol

This document describes the Limine v3 boot protocol interface used by the Kyronix kernel. It is the child of [Boot](sys-arch/kernel/boot/index.md).

## Overview

The Kyronix kernel uses the Limine v3 boot protocol to obtain boot-time information from the bootloader. The protocol uses a request/response mechanism where the kernel declares requests in a special ELF section (`.limine_requests`), and the bootloader populates the response fields before jumping to the kernel entry point.

## Request Types

| Request | Structure | Purpose |
|---|---|---|
| `LIMINE_FRAMEBUFFER_REQUEST` | `limine_framebuffer_response` | Linear framebuffer information |
| `LIMINE_MEMMAP_REQUEST` | `limine_memmap_response` | Physical memory map |
| `LIMINE_HHDM_REQUEST` | `limine_hhdm_response` | Higher Half Direct Map offset |
| `LIMINE_MODULE_REQUEST` | `limine_module_response` | Bootloader modules (initrd) |
| `LIMINE_KERNEL_ADDRESS_REQUEST` | `limine_kernel_address_response` | Kernel physical/virtual addresses |
| `LIMINE_RSDP_REQUEST` | `limine_rsdp_response` | ACPI RSDP physical address |
| `LIMINE_SMP_REQUEST` | `limine_smp_response` | SMP CPU information |

## Memory Map Types

| Type | Value | Description |
|---|---|---|
| `LIMINE_MEMMAP_USABLE` | 0 | Available for kernel use |
| `LIMINE_MEMMAP_RESERVED` | 1 | Reserved by hardware/firmware |
| `LIMINE_MEMMAP_ACPI_RECLAIMABLE` | 2 | Usable after ACPI parsing |
| `LIMINE_MEMMAP_ACPI_NVS` | 3 | ACPI NVS memory (must not reclaim) |
| `LIMINE_MEMMAP_BAD_MEMORY` | 4 | Defective memory region |
| `LIMINE_MEMMAP_BOOTLOADER_RECLAIMABLE` | 5 | Usable after bootloader exits |
| `LIMINE_MEMMAP_KERNEL_AND_MODULES` | 6 | Kernel and module images |
| `LIMINE_MEMMAP_FRAMEBUFFER` | 7 | Linear framebuffer memory |

## HHDM Response

The `limine_hhdm_response` provides the `offset` field, which is the base address of the Higher Half Direct Map. All physical addresses can be converted to virtual via `phys_to_virt(phys)` = `phys + g_hhdm_offset`.

## Kernel Address Response

The `limine_kernel_address_response` provides `physical_base` and `virtual_base`, used to compute the physical end of the kernel image for PMM initialization.

## Module Response

The `limine_module_response` provides an array of `limine_file` structures. Kyronix uses the first module as the initrd (CPIO archive).

## SMP Response

The `limine_smp_response` provides `cpu_count` and an array of `limine_smp_info` structures. Each entry contains `processor_id`, `lapic_id`, and `goto_address` (trampoline entry point for APs).

## Request Declaration

Requests are declared as `static volatile` structures bracketed by `LIMINE_REQUESTS_START_MARKER` and `LIMINE_REQUESTS_END_MARKER`. The base revision is set via `LIMINE_BASE_REVISION(3)`.

```c
LIMINE_REQUESTS_START_MARKER;

LIMINE_BASE_REVISION(3);

static volatile struct limine_memmap_request mmap_req = {
    .id = LIMINE_MEMMAP_REQUEST,
    .revision = 0,
    .response = NULL,
};

LIMINE_REQUESTS_END_MARKER;
```

IMPORTANT: The `response` field must be initialized to `NULL`. The bootloader sets it to a valid pointer if the request is fulfilled.

Last reviewed: 2026-07-22
