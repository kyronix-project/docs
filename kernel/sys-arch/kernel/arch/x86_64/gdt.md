# GDT

This document describes the Global Descriptor Table (GDT) setup in the Kyronix kernel.

## Overview

The GDT defines memory segments for kernel and user mode. Kyronix uses flat-model 64-bit segments (base=0, limit=max) as is standard for x86-64 long mode.

## GDT Layout

| Selector | Name | Description |
|----------|------|-------------|
| 0x00 | Null | Null descriptor (required) |
| 0x08 | Kernel Code | 64-bit ring 0, execute/read |
| 0x10 | Kernel Data | 64-bit ring 0, read/write |
| 0x18 | User Data | 64-bit ring 3, read/write |
| 0x20 | User Code | 64-bit ring 3, execute/read |
| 0x28+ | TSS | Task State Segments (one per CPU, 16 bytes each) |

## Segment Descriptors

Hardcoded 64-bit segment descriptors:

- Kernel Code: `0x00AF9A000000FFFF` -- present, 64-bit, ring 0, execute/read
- Kernel Data: `0x00CF92000000FFFF` -- present, 64-bit, ring 0, read/write
- User Data: `0x00CFF2000000FFFF` -- present, 64-bit, ring 3, read/write
- User Code: `0x00AFFA000000FFFF` -- present, 64-bit, ring 3, execute/read

## Task State Segment (TSS)

One TSS per CPU. Contains:

- `rsp0` -- kernel stack pointer for ring-3 transitions (set via `gdt_set_kernel_stack`)
- `ist[0]` -- Double Fault IST stack (16384 bytes)
- `ist[1]` -- NMI IST stack (16384 bytes)

## Initialization

`gdt_init()` performs the following steps:

1. Populate GDT entries (null, kernel code/data, user code/data, TSS for BSP)
2. Load GDT via `lgdt`
3. Far jump to reload CS with selector 0x08
4. Load DS/ES/SS with 0x10 (kernel data)
5. Zero FS/GS
6. Load TSS via `ltr`

## Per-CPU Setup

`gdt_ap_load(cpu_id)` creates per-CPU TSS entries and reloads GDT+TSS for Application Processors. Each CPU gets its own TSS with a dedicated double-fault and NMI IST stack.
