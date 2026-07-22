# IRQ Handling

This document describes the interrupt handling implementation in the Kyronix kernel. It is the child of [Kernel Implementation Notes](index.md).

The Kyronix kernel dispatches interrupts through a common assembly entry point (`isr_common`) that supports PIC IRQs, CPU exceptions, LAPIC interrupts, and system calls.

## PIC Remapping

The Programmable Interrupt Controller (PIC) is remapped to interrupt vectors 32–47 to avoid collision with CPU exception vectors 0–31.

## ISR Common Entry Point

The `isr_common` assembly stub performs the following steps:

1. If the interrupt originated from ring 3 (user mode), execute `swapgs` to switch to the kernel GS base.
2. Push all General-Purpose Registers (GPRs) onto the kernel stack.
3. Call the C dispatch function `isr_dispatch`.
4. Pop all GPRs from the kernel stack.
5. If returning to ring 3, execute `swapgs` to restore the user GS base.
6. Execute `iretq` to return from the interrupt.

## Interrupt Dispatch

`isr_dispatch` handles the following interrupt sources:

### CPU Exceptions (Vectors 0–31)

- **User-mode exception:** The exception is delivered as a signal to the faulting process.
- **Kernel-mode exception:** The kernel invokes a panic handler with a stack backtrace.

### Page Fault (Vector 14)

Page faults are handled with demand paging for the following cases:

1. **User stack growth:** Faults within `USER_STACK_GROW_BASE` to `USER_STACK_TOP` trigger automatic stack expansion.
2. **VMA (Virtual Memory Area) regions:** Faults within a mapped VMA region trigger demand paging via `vmm_user_range_fault_in`.

### PIC IRQs (Vectors 32–47)

PIC IRQs are forwarded from the PIC to the corresponding vector offset.

### System Call (Vector 0x80)

System calls are dispatched via the `syscall` instruction (vector 128).

### LAPIC Timer (Vector 224)

The LAPIC (Local Advanced Programmable Interrupt Controller) timer interrupt is handled with the same preemption logic as PIC IRQ 0.

### LAPIC Spurious (Vector 255)

Spurious LAPIC interrupts are acknowledged without further processing.

## Timer Tick (PIC IRQ 0)

Each PIC IRQ 0 tick performs the following operations:

1. Increment the global tick counter (`g_ticks`).
2. Update the cursor blink state.
3. Reap zombie threads.
4. Poll the network stack.
5. Process the timer mask for sleep/wake operations.
6. Invoke the scheduler for preemption.

## Double Fault

Double faults (vector 8) use Interrupt Stack Table (IST) stack 1, backed by a dedicated 16 KiB stack, to ensure a usable stack is available when the primary stack is corrupted.

## Non-Maskable Interrupt (NMI)

Non-Maskable Interrupts (NMIs, vector 2) use IST stack 2, backed by a dedicated 16 KiB stack, to handle hardware-critical conditions independently of normal interrupt processing.

## Kernel Backtrace

The kernel backtrace routine scans the kernel stack for return addresses within the kernel text region defined by the bounds:

```
[0xffffffff80000000, 0xffffffff80040000)
```

Any pointer found within this range is reported as a valid kernel return address.

Last reviewed: 2026-07-22
