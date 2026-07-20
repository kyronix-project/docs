# IRQ Handling

This document describes the interrupt handling implementation in the Kyronix kernel.

## Interrupt Flow

1. Hardware interrupt or exception occurs
2. CPU looks up the IDT entry and jumps to the ISR stub
3. ISR stub (`idt_stubs.S`) saves registers and calls `isr_dispatch()`
4. `isr_dispatch()` in `idt.c` handles the event
5. On return, registers are restored and `iretq` returns to the interrupted context

## ISR Stub Details

The common ISR handler (`isr_common`) performs:

1. Check CS RPL (was this from user mode?)
2. If from user mode, execute `swapgs` (switch to kernel GS base)
3. Push all 15 general-purpose registers (r15..rax) forming a `cpu_state_t`
4. Push interrupt number and error code
5. Call `isr_dispatch(cpu_state_t *state)` with RSP as argument
6. On return, pop all registers
7. If from user mode, execute `swapgs`
8. Execute `iretq`

## Dispatch Logic

`isr_dispatch()` handles:

### Exceptions (0-31)

| Vector | Handler |
|--------|---------|
| 14 (Page Fault) | Demand paging, stack growth, then SIGSEGV/SIGBUS |
| 3 (#BP) | Debug breakpoint (ptrace stop) |
| 1 (#TF) | Single step (ptrace stop) |
| 0-31 (user mode) | Send signal (SIGSEGV, SIGFPE, SIGILL, SIGTRAP, SIGBUS) |
| 0-31 (kernel mode) | Kernel panic with register dump and backtrace |

### Hardware IRQs (32-47)

- **IRQ 0 (PIT, vector 32):** Tick counter, cursor blink, timer wakeups, preemption
- **Other IRQs:** Dispatch to registered handler via `g_irq_handlers[irq]`

### LAPIC Vectors

- **0xFF (spurious):** Ignored
- **0xE0 (LAPIC timer):** Same preemption logic as PIT IRQ 0

## IRQ Handler Registration

Drivers register interrupt handlers via:

```c
g_irq_handlers[irq] = handler_function;
```

The handler is called with the IRQ number as argument.

## Page Fault Handling

The page fault handler (vector 14) performs:

1. Read faulting address from CR2
2. Check if the fault is in a user VMA region
3. If stack growth is needed (fault near USER_STACK_TOP), grow the stack
4. If demand paging is enabled (VMA with `free_on_unmap`), allocate and map a page
5. If none of the above, send SIGSEGV (user mode) or panic (kernel mode)

### Stack Growth

The kernel supports automatic stack growth within the range `USER_STACK_GROW_BASE` to `USER_STACK_TOP`. A fault in the guard region (up to 4 pages below the current stack) triggers allocation of a new stack page.
