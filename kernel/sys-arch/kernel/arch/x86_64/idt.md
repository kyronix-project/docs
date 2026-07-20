# IDT

This document describes the Interrupt Descriptor Table (IDT) setup and interrupt handling in the Kyronix kernel.

## Overview

The IDT is a 256-entry table, 16-byte aligned. Each entry points to an ISR stub that saves the full register state and dispatches to a C handler.

## Vector Assignments

| Vector | Purpose |
|--------|---------|
| 0-31 | CPU exceptions (ISRs 0-31) |
| 32-47 | Hardware IRQs (PIC remapped) |
| 0x80 | System call gate (user-callable, ring 3) |
| 0xE0 | LAPIC timer |
| 0xFF | LAPIC spurious |

## PIC Remapping

The 8259 PIC is remapped to vectors 32-47 via `pic_remap(32, 40)`. All IRQs are initially masked. The PIC driver provides:

- `pic_remap(off1, off2)` -- ICW1-ICW4 initialization sequence
- `pic_mask_all()` -- mask all IRQs
- `pic_mask_irq(irq)` / `pic_unmask_irq(irq)` -- individual IRQ control
- `pic_send_eoi(irq)` -- send End-Of-Interrupt

## Exception Handling

Exceptions (vectors 0-31) are handled by `isr_dispatch()`:

- **Page fault (14):** Checks for demand paging (stack growth, VMA lazy allocation) before signaling
- **User mode:** Sends appropriate signal (SIGSEGV, SIGBUS, SIGFPE, SIGILL, SIGTRAP)
- **Kernel mode:** Kernel panic with full register dump and optional backtrace
- **#BP/#TF with ptrace:** Stops process for debugger

## Hardware IRQ Handling

IRQ 0 (PIT) performs:

- Increment global tick counter (`g_ticks`)
- Framebuffer cursor blink
- Process reaping
- Network polling
- Per-process timer wakeups (sleep/alarm/itimer)
- Preemption of userspace processes via scheduler

Other IRQs dispatch to registered handlers via `g_irq_handlers[irq]`.

## ISR Stubs

The ISR stubs (`idt_stubs.S`) use two macros:

- `isr_no_err` -- pushes a dummy 0 error code (for exceptions that don't push one)
- `isr_err` -- CPU pushes the error code automatically

The common handler (`isr_common`):

1. Checks CS RPL, does `swapgs` if coming from ring 3
2. Pushes all 15 GP registers forming a `cpu_state_t` frame
3. Calls `isr_dispatch(cpu_state_t *state)`
4. On return, pops all registers, `swapgs` if needed, `iretq`

## IST Stacks

Two interrupt stacks are configured via the TSS:

- IST index 1: Double Fault stack (16384 bytes)
- IST index 2: NMI stack (16384 bytes)
