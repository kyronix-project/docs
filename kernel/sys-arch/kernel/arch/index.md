# Architecture

This document describes the hardware abstraction layer for the Kyronix kernel. It is the child of [Kernel](sys-arch/kernel/index.md) and parent of architecture-specific documents.

## Supported Architectures

The Kyronix kernel currently supports x86_64 (AMD64) only. The architecture-specific code lives in `kernel/arch/x86_64/`.

## Architecture Components

| Component | Source | Description |
|---|---|---|
| CPU Primitives | `arch/x86_64/cpu.h` | I/O ports, MSRs, control registers, compiler attributes |
| GDT | `arch/x86_64/gdt.c` | Global Descriptor Table and Task State Segment |
| IDT | `arch/x86_64/idt.c` | Interrupt Descriptor Table and ISR dispatch |
| LAPIC | `arch/x86_64/lapic.c` | Local APIC MMIO, IPI, timer calibration |
| PIT | `arch/x86_64/pit.c` | Programmable Interval Timer and RTC |
| Syscall Setup | `arch/x86_64/syscall_setup.c` | SYSCALL/SYSRET MSRs, SSE, per-CPU data |
| Syscall Entry | `arch/x86_64/syscall_entry.S` | SYSCALL entry and userspace trampolines |
| IDT Stubs | `arch/x86_64/idt_stubs.S` | Assembly ISR entry/exit stubs |

Last reviewed: 2026-07-22
