# Scheduler

This document describes the preemptive scheduler in the Kyronix kernel.

## Overview

Kyronix uses a simple, preemptive round-robin scheduler with per-CPU idle processes. The scheduler is driven by timer interrupts (PIT and LAPIC timer) at approximately 250 Hz.

## Scheduling Algorithm

The scheduler uses a 64-bit bitmask (`g_ready_mask`) to track which process slots are ready to run. The scheduling decision is:

1. Scan the bitmask from the last-scheduled position (round-robin)
2. Find the first ready process that is not the current process
3. Atomically transition the process from `PROC_READY` to `PROC_RUNNING` via CAS
4. Context-switch to the selected process

## Context Switch

The context switch is performed by `sched_switch()` (implemented in `sched.S`), which:

1. Saves the current process's callee-saved registers and FPU state
2. Loads the next process's callee-saved registers and FPU state
3. Switches the kernel stack pointer
4. Switches the address space (CR3) if different
5. Updates per-CPU data (current process, file descriptor table)

## Preemption

Preemption occurs on timer interrupts (IRQ 0 or LAPIC timer). The interrupt handler checks:

1. Is the interrupted context a userspace process?
2. Is the process in `PROC_RUNNING` state?
3. Is there another ready process?

If all conditions are met, the scheduler picks the next ready process and context-switches.

## Blocking

`sched_yield_blocking()` is used when a process needs to block:

1. Set process state to `PROC_WAITING`
2. Clear the ready bit
3. Pick the next ready process (or the CPU's idle process)
4. Context-switch to the selected process
5. On wakeup, restore `PROC_RUNNING` state

## Idle Process

Each CPU has an idle process that runs when no other process is ready. The idle process runs with interrupts enabled and halts the CPU (`sti; hlt`) to save power.

## SMP Scheduling

Each CPU independently schedules from the shared `g_ready_mask`. The round-robin position is tracked per-CPU (`g_last_scheduled[cpu_id]`) to reduce contention.

## Timers

The following per-process timers are checked on each tick:

| Timer | Description |
|-------|-------------|
| `wakeup_tick` | nanosleep/clock_nanosleep wakeup time |
| `alarm_tick` | POSIX alarm() expiry |
| `itimer_value_ms` / `itimer_interval_ms` | POSIX interval timer (ITIMER_REAL) |
