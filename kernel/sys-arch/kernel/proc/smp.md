# SMP

This document describes the Symmetric Multi-Processing (SMP) support in the Kyronix kernel.

## Overview

Kyronix supports multiple CPUs via the Limine SMP protocol. The Boot Strap Processor (BSP) initializes the system, then boots Application Processors (APs) to participate in scheduling.

## CPU Limit

The kernel supports up to 16 CPUs (`MAX_CPUS`). Each CPU has a dedicated `cpu_local_t` structure accessed via the GS segment register.

## Per-CPU Data

| Offset | Field | Description |
|--------|-------|-------------|
| GS:0 | `kernel_rsp` | Kernel stack pointer |
| GS:8 | `user_rsp` | Saved user stack pointer |
| GS:16 | `cpu_id` | CPU identifier |
| GS:20 | `lapic_id` | Local APIC ID |
| GS:32 | `current` | Current process pointer |
| GS:48 | `current_space` | Current address space |
| GS:56 | `g_fds_ptr` | File descriptor table |
| GS:64 | `reap_thread` | Deferred thread reap |

## Initialization

### BSP Setup

1. `smp_init()` reads the Limine SMP response to discover all CPUs
2. Assigns CPU IDs (BSP = 0, APs = 1, 2, ...)
3. Stores `cpu_local_t` pointers in the Limine extra_argument field

### AP Boot

1. `smp_boot_aps()` creates idle processes for each AP
2. Sets the `goto_address` of each AP to `ap_trampoline`
3. The trampoline code (`ap_trampoline.S`) initializes each AP:
   - Loads per-CPU GDT and IDT
   - Sets GS base to `cpu_local_t`
   - Enables SSE
   - Configures SYSCALL MSRs
   - Initializes LAPIC
   - Waits for `g_kernel_ready` flag
   - Starts LAPIC timer at 250 Hz
   - Enters `ap_sched_loop()`

## AP Scheduling Loop

Each AP runs `ap_sched_loop()`:

1. Claim the next ready process from the shared ready mask
2. If a process is found, context-switch to it
3. If no process is ready, halt with interrupts enabled (`sti; hlt`)
4. On return from context-switch, restore idle process state

## Synchronization

- `g_ready_mask` is accessed via atomic OR/AND operations
- `g_aps_ready` uses atomic increment with `__ATOMIC_RELEASE` ordering
- `g_kernel_ready` uses atomic load with `__ATOMIC_ACQUIRE` ordering
- Process state transitions use `__sync_bool_compare_and_swap` (CAS)
