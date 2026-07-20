# Scheduling

This document describes the internals of the Kyronix scheduler.

## Context Switch

The context switch is implemented in assembly (`sched.S`) and performs:

### Process Switch (`sched_switch(next)`)

1. Save callee-saved registers (rbx, rbp, r12-r15) of the current process
2. Save FPU state (FXSAVE) of the current process
3. Switch kernel stack pointer (`rsp`) to the next process's `kstack_rsp`
4. Restore FPU state (FXRSTOR) of the next process
5. Restore callee-saved registers of the next process
6. If the address space changed, load CR3 with the new PML4
7. Return (which restores RIP from the new stack)

### Per-CPU State Update

After context switch, the following per-CPU state is updated:

```c
vfs_set_fdtable(next->fds);         // switch FD table
g_current_space = next->space;      // switch address space
cpu_set_kernel_stack(next->kstack_top); // update TSS rsp0
```

## Ready Mask

The scheduler uses a 64-bit atomic bitmask (`g_ready_mask`) to track ready processes:

- Bit N = 1: process slot N is ready to run
- Bit N = 0: process slot N is not ready

`proc_next_ready()` scans the bitmask from the last-scheduled position using `__builtin_ctzll` to find the next set bit.

## Preemption

Preemption occurs on timer interrupts (IRQ 0 or LAPIC timer vector 0xE0):

1. Timer interrupt fires
2. Increment `g_ticks`
3. Check if interrupted context is a userspace process
4. If yes, call `proc_next_ready()` to find another ready process
5. If a different process is found, context-switch to it

## Blocking

`sched_yield_blocking()` is called when a process needs to wait:

1. Set state to `PROC_WAITING`, clear ready bit
2. Claim the next ready process (or idle)
3. Context-switch
4. On wakeup, restore `PROC_RUNNING`

## Idle Loop

Each CPU's idle process runs:

```c
for (;;) {
    sti();   // enable interrupts
    hlt();   // halt until interrupt
}
```

This allows the CPU to enter a low-power state while waiting for work.

## Process Reaping

Dead processes are reaped via `proc_defer_thread_reap()`. The reaped process's reference count is decremented on the next timer tick, preventing stack-use-after-free.
