# Scheduler

This document describes the scheduler in the Kyronix kernel. It is the child of [Process Management](sys-arch/kernel/proc/index.md).

## Source

`kernel/proc/proc.c`, `kernel/proc/sched.S`

## Algorithm

The scheduler implements **per-CPU round-robin with lock-free bitmap scanning**. The `g_ready_mask` 64-bit bitmask enables O(1) next-process selection via `__builtin_ctzll` (count trailing zeros).

## Key Functions

| Function | Description |
|---|---|
| `proc_next_ready(skip)` | Find next ready process via bitmap scan |
| `sched_claim_next(skip)` | CAS-based claim (READY -> RUNNING) |
| `sched_yield_blocking()` | Voluntary yield on block |
| `sched_switch(next)` | Context switch (assembly) |
| `proc_idle_until_ready(skip)` | Busy-wait for ready process |
| `proc_create_idle(cpu_id, entry)` | Create idle process for a CPU |

## Context Switch (`sched_switch`)

The context switch in `sched.S` performs:

1. Save callee-saved registers (RBX, RBP, R12-R15)
2. Save FPU/SSE state via `fxsave64`
3. Save FS base via `rdmsr(IA32_FS_BASE)`
4. Save kernel stack pointer (`kstack_rsp`) and user RSP
5. Load next process's kernel stack, FS base, user RSP
6. Switch address space via `vmm_switch(next->space)` (CR3)
7. Restore FPU/SSE state via `fxrstor64`
8. Pop callee-saved registers and return

## SMP Scheduling

Each CPU runs its own scheduling loop (`ap_sched_loop`):

1. Try `sched_claim_next(idle)` -- CAS from READY to RUNNING
2. If found: switch from idle to the claimed process
3. If not found: `sti; hlt` (halt until next interrupt)

Preemption occurs on both PIC IRQ 0 and LAPIC timer tick (vector 224). If a higher-priority process is found via `sched_claim_next`, the current process is preempted.

## Kernel Stack Layout

Each process gets 16 pages (64 KiB) of kernel stack plus 1 guard page:

- Guard page: unmapped (detects overflow)
- Usable stack: 16 pages mapped with `VMM_KDATA`
- Virtual base: `0xffff920000000000` (bump-allocated)

## FPU State

Each process has 512 bytes of FPU/SSE state (`fpu_state`) at offset 3328, saved/restored on every context switch via `fxsave64`/`fxrstor64`. Initialized with FCW=`0x037F` and MXCSR=`0x1F80` (all exceptions masked).

Last reviewed: 2026-07-22
