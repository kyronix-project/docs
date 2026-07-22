# GDT

This document describes the Global Descriptor Table (GDT) implementation in the Kyronix kernel. It is the child of [x86-64 Architecture](sys-arch/kernel/arch/x86_64/index.md).

## Source

`kernel/arch/x86_64/gdt.c`

## Overview

The GDT provides segment descriptors for kernel and user code/data, plus per-CPU Task State Segment (TSS) descriptors. In long mode, segments are mostly flat (base=0, limit=4 GiB), but the TSS is essential for interrupt stack switching and I/O permission bitmap.

## GDT Structure

```c
typedef struct {
    gdt_entry_t entries[5];         // null, kcode, kdata, udata, ucode
    tss_desc_t  tss[MAX_CPUS];     // one TSS descriptor per CPU
} gdt_t;  // aligned(16)
```

## Segment Values

| Entry | Selector | Encoded Value | Meaning |
|---|---|---|---|
| Kernel code | `0x08` | `0x00AF9A000000FFFF` | 64-bit, ring 0, executable, readable |
| Kernel data | `0x10` | `0x00CF92000000FFFF` | 64-bit, ring 0, writable |
| User data | `0x18` | `0x00CFF2000000FFFF` | 64-bit, ring 3, writable |
| User code | `0x20` | `0x00AFFA000000FFFF` | 64-bit, ring 3, executable, readable |

## TSS Structure

The TSS (104 bytes on x86-64) provides:

- **rsp0**: Ring 0 stack pointer, used on privilege level transitions (interrupts from ring 3)
- **ist[7]**: Interrupt Stack Table entries for critical handlers (Double Fault, NMI)
- **iopb_offset**: Offset to I/O permission bitmap

```c
typedef struct {
    uint32_t reserved0;
    uint64_t rsp0, rsp1, rsp2;
    uint64_t reserved1;
    uint64_t ist[7];
    uint64_t reserved2;
    uint16_t reserved3;
    uint16_t iopb_offset;  // = sizeof(tss_t)
} tss_t;  // 104 bytes, packed
```

## Key Functions

| Function | Description |
|---|---|
| `gdt_init()` | Initialize GDT, BSP TSS, load GDT and TSS on BSP |
| `gdt_ap_load(cpu_id)` | Load GDT and TSS on an Application Processor |
| `gdt_set_kernel_stack(rsp0)` | Update TSS rsp0 for the current CPU |
| `tss_init(tss)` | Initialize TSS fields (IST stacks, IOPB offset) |

## IST Stacks

| IST Index | Stack | Size | Assigned To |
|---|---|---|---|
| 1 | `g_ist_df` | 16 KiB | Double Fault (#8) |
| 2 | `g_ist_nmi` | 16 KiB | NMI (vector 2) |

NOTE: IST stacks provide dedicated, non-overflowable stacks for the most critical exceptions, preventing kernel stack corruption during Double Faults.

Last reviewed: 2026-07-22
