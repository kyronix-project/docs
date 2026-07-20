# LAPIC

This document describes the Local APIC (Advanced Programmable Interrupt Controller) driver in the Kyronix kernel.

## Overview

The Local APIC handles:

- Per-CPU timer interrupts
- Inter-Processor Interrupts (IPIs)
- Spurious interrupt handling

## MMIO Mapping

The LAPIC is memory-mapped at physical address read from `IA32_APIC_BASE` MSR (0x1B). It is mapped to virtual address `0xfffffe0000000000` with `VMM_PCD` (uncacheable) during initialization.

## Registers

The driver defines all standard LAPIC MMIO register offsets:

| Register | Offset | Description |
|----------|--------|-------------|
| ID | 0x020 | APIC ID |
| VERSION | 0x030 | APIC version |
| TPR | 0x080 | Task Priority Register |
| EOI | 0x0B0 | End-Of-Interrupt |
| SVR | 0x0F0 | Spurious Vector Register |
| ICR_LO/HI | 0x300/0x310 | Interrupt Command Register |
| LVT Timer | 0x320 | Timer LVT entry |

## Initialization

`lapic_init()` performs:

1. Read `IA32_APIC_BASE` MSR
2. Enable LAPIC if not already enabled
3. Map MMIO region into kernel space
4. Enable SVR with spurious vector 0xFF
5. Mask error/thermal/perf LVT entries
6. Set TPR to 0 (accept all interrupts)
7. Log LAPIC ID and version

## Timer

### Calibration

`lapic_calibrate_timer()` uses PIT Channel 0 as a reference:

1. Set LAPIC timer to max count (0xFFFFFFFF) with divider 0x0B
2. Wait for 5 PIT counter wraps
3. Compute ticks/ms from the calibrated frequency

### Periodic Mode

`lapic_timer_start_periodic(hz)` configures the LAPIC timer in periodic mode at the specified frequency (default 250 Hz).

## IPI Delivery

`lapic_send_ipi(lapic_id, icr_lo)` sends an Inter-Processor Interrupt:

1. Wait for ICR not pending
2. Write destination LAPIC ID to ICR_HI
3. Write delivery mode to ICR_LO
4. Wait for delivery confirmation

## EOI

`lapic_eoi()` writes 0 to the EOI register to acknowledge the current interrupt.
