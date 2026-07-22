# x86-64 Architecture

This document describes the x86-64 architecture support in the Kyronix kernel. It is the child of [Architecture](sys-arch/kernel/arch/index.md) and parent of x86-64-specific documents.

## Files

| File | Purpose |
|---|---|
| `cpu.h` | CPU primitives, I/O ports, MSRs, control registers, data structures |
| `gdt.c` | GDT creation, TSS initialization, per-CPU segments |
| `idt.c` | IDT setup, PIC remapping, ISR dispatch |
| `idt_stubs.S` | Assembly ISR entry/exit stubs, isr_stub_table |
| `lapic.c` | Local APIC initialization, IPI, timer calibration |
| `pit.c` | PIT channel 0 programming, RTC epoch reading |
| `syscall_setup.c` | SYSCALL/SYSRET configuration, SSE enable, per-CPU local data |
| `syscall_entry.S` | SYSCALL entry point, enter_userspace trampolines |

## GDT Layout

| Selector | Entry | Description |
|---|---|---|
| `0x00` | Null | Null descriptor |
| `0x08` | Kernel code | 64-bit ring 0 executable |
| `0x10` | Kernel data | 64-bit ring 0 writable |
| `0x18` | User data | 64-bit ring 3 writable |
| `0x20` | User code | 64-bit ring 3 executable |
| `0x28 + n*0x10` | TSS for CPU n | Task State Segment |

## IDT Vector Layout

| Vectors | Source | Gate Type | Description |
|---|---|---|---|
| 0-31 | CPU exceptions | INT_GATE | #DE through #SX |
| 32-47 | PIC IRQ 0-15 | INT_GATE | Legacy PIC interrupts |
| 0x80 (128) | SYSCALL | USER_GATE (DPL=3) | System call entry |
| 224 (0xE0) | LAPIC timer | INT_GATE | Per-CPU timer tick |
| 255 (0xFF) | LAPIC spurious | INT_GATE | Spurious interrupt |

## IST Usage

| IST Index | Stack | Assigned To |
|---|---|---|
| 1 | 16 KiB dedicated | Double Fault (#8) |
| 2 | 16 KiB dedicated | NMI (vector 2) |

## Per-CPU Data

Per-CPU data is accessed via the GS segment register. `MSR_GS_BASE` points to `cpu_local_t` for the current CPU. Key offsets:

| Offset | Field | Description |
|---|---|---|
| 0 | `kernel_rsp` | Kernel stack pointer (for SYSCALL entry) |
| 8 | `user_rsp` | User stack pointer (saved on SYSCALL) |
| 16 | `cpu_id` | CPU identifier |
| 32 | `current` | Current `proc_t` pointer |
| 40 | `idle` | Idle process pointer |

The `swapgs` instruction swaps between user and kernel GS bases on privilege transitions.

Last reviewed: 2026-07-22
