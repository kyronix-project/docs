# LAPIC

This document describes the Local APIC implementation in the Kyronix kernel. It is the child of [x86-64 Architecture](sys-arch/kernel/arch/x86_64/index.md).

## Source

`kernel/arch/x86_64/lapic.c`

## Overview

The Local APIC (Advanced Programmable Interrupt Controller) provides per-CPU interrupt handling, Inter-Processor Interrupts (IPIs), and the LAPIC timer. The MMIO registers are mapped at virtual address `0xfffffe0000000000`.

## Initialization

1. Read `IA32_APIC_BASE` MSR to get physical MMIO address
2. Enable LAPIC if disabled (set `IA32_APIC_BASE_ENABLE`)
3. Map physical LAPIC to `LAPIC_VIRT` with `VMM_KDATA | VMM_PCD` (page-cache disabled for MMIO)
4. Enable Spurious Vector Register (SVR) with spurious vector
5. Mask error, thermal, performance, and timer LVT entries
6. Clear Task Priority Register (TPR)
7. Read LAPIC ID and version

## Key Functions

| Function | Description |
|---|---|
| `lapic_init()` | Full LAPIC initialization and MMIO mapping |
| `lapic_eoi()` | Write 0 to EOI register (end-of-interrupt) |
| `lapic_send_ipi(lapic_id, icr_lo)` | Send IPI to specific LAPIC |
| `lapic_send_ipi_self(icr_lo)` | Send self-IPI |
| `lapic_calibrate_timer()` | Calibrate LAPIC timer against PIT |
| `lapic_timer_start_periodic(hz)` | Start periodic timer at given frequency |
| `lapic_timer_freq()` | Return calibrated timer frequency (ticks/ms) |

## Timer Calibration

The calibration algorithm:

1. Set LAPIC timer to one-shot mode with divisor `0x0B` and initial count `0xFFFFFFFF`
2. Count 5 PIT counter wraps (each wrap = one PIT period)
3. Compute `remaining = 0xFFFFFFFF - current_count`
4. Derive `g_lapic_timer_freq = remaining / 5` (ticks per millisecond)

The LAPIC timer is then started in periodic mode at 250 Hz for scheduling ticks.

## IPI Mechanism

Inter-Processor Interrupts are sent via the Interrupt Command Register (ICR):

1. Wait for send-pending bit to clear
2. Write target LAPIC ID to ICR_HI
3. Write delivery info to ICR_LO
4. Wait for send-pending to clear again

## Register Layout

| Offset | Register | Description |
|---|---|---|
| 0x20 | TPR | Task Priority Register |
| 0x80 | EOI | End of Interrupt |
| 0xB0 | ICR_LO | Interrupt Command (low) |
| 0xC0 | ICR_HI | Interrupt Command (high) |
| 0xD0 | SVR | Spurious Vector Register |
| 0x320 | LVT Timer | Timer LVT entry |
| 0x350 | LVT LINT0 | LINT0 LVT entry |
| 0x360 | LVT LINT1 | LINT1 LVT entry |
| 0x370 | LVT Error | Error LVT entry |

Last reviewed: 2026-07-22
