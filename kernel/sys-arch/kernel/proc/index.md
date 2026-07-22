# Process Management

This document describes the process management subsystem of the Kyronix kernel. It is the child of [Kernel](sys-arch/kernel/index.md) and parent of process management component documents.

## Components

| Component | Source | Description |
|---|---|---|
| Process Table | `proc/proc.c` | Process allocation, lifecycle, scheduling |
| Scheduler | `proc/sched.S` | Context switch (assembly) |
| SMP | `proc/smp.c` | CPU enumeration, AP boot |
| Signals | `proc/signal.c` | POSIX signal delivery |
| Jails | `proc/jail.c` | Sandbox/container isolation |
| ELF Loader | `exec/elf.c` | 64-bit ELF parsing and loading |
| Process Exec | `exec/process.c` | Exec, stack setup, userspace entry |

## Process States

| State | Value | Description |
|---|---|---|
| `PROC_UNUSED` | 0 | Slot is free |
| `PROC_RUNNING` | 1 | Currently executing on a CPU |
| `PROC_READY` | 2 | Eligible to run, waiting for time slice |
| `PROC_WAITING` | 3 | Blocked (I/O, signal, etc.) |
| `PROC_ZOMBIE` | 4 | Exited, not yet reaped |
| `PROC_DYING` | 5 | In the process of exiting |
| `PROC_STOPPED` | 6 | Job-control stopped |

## Process Table

- Fixed array of 64 slots (`PROC_MAX`)
- Spinlock-protected allocation
- PID = slot index + 1 (PID 0 reserved for idle)

## Global Bitmasks

| Bitmask | Description |
|---|---|
| `g_ready_mask` | Processes in READY state (eligible for scheduling) |
| `g_used_mask` | All allocated process slots |
| `g_timer_mask` | Processes with active timers |

Last reviewed: 2026-07-22
