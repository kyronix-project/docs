# IDT

This document describes the Interrupt Descriptor Table (IDT) implementation in the Kyronix kernel. It is the child of [x86-64 Architecture](sys-arch/kernel/arch/x86_64/index.md).

## Source

`kernel/arch/x86_64/idt.c`, `kernel/arch/x86_64/idt_stubs.S`

## Overview

The IDT provides 256 interrupt vector entries. The Kyronix kernel maps CPU exceptions (vectors 0-31), legacy PIC IRQs (vectors 32-47), the SYSCALL vector (0x80), LAPIC timer (224), and LAPIC spurious (255) through a table of 51 assembly stubs (`isr_stub_table`).

## Vector Allocation

| Vectors | Source | Gate Type | IST | Handler |
|---|---|---|---|---|
| 0-31 | CPU exceptions | INT_GATE | -- | `isr_dispatch()` |
| 3 (#BP) | Breakpoint | TRAP_GATE | -- | Allows debugger resume |
| 8 (#DF) | Double Fault | INT_GATE | IST 1 | Dedicated DF stack |
| 2 (NMI) | NMI | INT_GATE | IST 2 | Dedicated NMI stack |
| 32-47 | PIC IRQ 0-15 | INT_GATE | -- | `isr_dispatch()` |
| 128 (0x80) | SYSCALL | USER_GATE (DPL=3) | -- | `isr_dispatch()` |
| 224 (0xE0) | LAPIC timer | INT_GATE | -- | `isr_dispatch()` |
| 255 (0xFF) | LAPIC spurious | INT_GATE | -- | Silently returns |

## Gate Types

| Constant | Value | Description |
|---|---|---|
| `IDT_INT_GATE` | `0x8E` | Interrupt gate: present, DPL=0, clears IF on entry |
| `IDT_TRAP_GATE` | `0x8F` | Trap gate: present, DPL=0, does NOT clear IF |
| `IDT_USER_GATE` | `0xEE` | Interrupt gate: present, DPL=3 (user-accessible) |

## ISR Dispatch (`isr_dispatch`)

The central C function called from `isr_common` assembly after register save. It handles:

### CPU Exceptions (vectors 0-31)

- **User-mode exceptions**: Deliver signal to process via `exception_signal(n)` (SIGFPE, SIGILL, SIGTRAP, SIGSEGV, SIGBUS, etc.)
- **Page fault (vector 14)**: `handle_user_page_fault()` implements demand paging for stack growth and VMA-based anonymous pages
- **Breakpoint/Debug (vectors 1, 3)**: Check `tracer_pid` for ptrace support; #BP decrements RIP by 1 past `int3`
- **Kernel-mode exceptions**: Full panic with register dump, exception name, CR2 (for #PF), and kernel backtrace

### PIC IRQs (vectors 32-47)

- **IRQ 0 (timer tick)**: Increments `g_ticks`, cursor blink, zombie reaping, network polling, timer mask processing (wakeup, alarm, itimer), preemption check
- **IRQs 1-15**: Dispatch to registered `g_irq_handlers[irq]` handlers

### Special Vectors

- **LAPIC timer (224)**: Same preemption logic as PIC IRQ 0
- **LAPIC spurious (255)**: Silently returns

## ISR Stub Assembly

The `isr_common` routine in `idt_stubs.S` performs:

1. `swapgs` if entering from ring 3 (privilege transition)
2. Push all 15 GPRs (forming `cpu_state_t` frame)
3. Call `isr_dispatch(state)` in C
4. Pop all GPRs
5. `swapgs` if returning to ring 3
6. `iretq`

## IRQ Registration

```c
void request_irq(uint8_t irq, irq_handler_fn fn, void *arg);
```

Registers a handler for PIC IRQ 0-15 and unmasks it via `pic_unmask_irq()`.

## Kernel Backtrace

The `kernel_backtrace()` function scans the kernel stack for return addresses in the range `[0xffffffff80000000, 0xffffffff80040000)` and prints up to 48 candidates. It is called only during kernel panics.

Last reviewed: 2026-07-22
