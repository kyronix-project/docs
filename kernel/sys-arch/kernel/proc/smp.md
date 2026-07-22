# SMP

This document describes the Symmetric Multi-Processing (SMP) support in the Kyronix kernel. It is the child of [Process Management](sys-arch/kernel/proc/index.md).

## Source

`kernel/proc/smp.c`, `kernel/proc/smp.h`, `kernel/proc/ap_trampoline.S`

## Overview

SMP support enables the kernel to run on multiple CPU cores. The BSP (Bootstrap Processor) discovers CPUs via the Limine SMP protocol and wakes APs (Application Processors) via a trampoline mechanism.

## CPU Discovery (`smp_init`)

1. Read `limine_smp_response` from bootloader
2. Set `g_cpu_count` to reported CPU count
3. Identify BSP by its LAPIC ID, assign CPU ID 0
4. Assign sequential IDs to APs (starting at 1)
5. Store `&g_cpu_local[cid]` in each CPU's `extra_argument`

## AP Boot Sequence (`smp_boot_aps`)

1. Create idle process for each non-BSP CPU
2. Write `ap_trampoline` to each CPU's `goto_address` (Limine wake-up mechanism)
3. Wait until `g_aps_ready == g_cpu_count - 1`

## AP Trampoline (`ap_trampoline`)

The AP entry point in assembly:

1. `cli` -- disable interrupts
2. Load `limine_smp_info.extra_argument` (points to `cpu_local_t`)
3. Load `cpu_local_t.current` (idle process)
4. Load `proc_t.kstack_top` as RSP
5. Call `ap_init_cpu(cpu_local_t *)` in C

## AP Initialization (`ap_init_cpu`)

Each AP performs full initialization (in order):

1. GDT: `gdt_ap_load(cpu_id)` -- per-CPU GDT + TSS
2. IDT: `idt_load_ap()` -- reload IDT
3. MSR setup: Write `IA32_GS_BASE` and `IA32_KERNEL_GS_BASE`
4. SSE: `cpu_enable_sse()`
5. SYSCALL MSRs: Configure SYSCALL/SYSRET (same as BSP)
6. LAPIC: Enable spurious vector, mask LVT entries
7. Signal readiness: Atomically increment `g_aps_ready`
8. Wait for BSP: Spin on `g_kernel_ready`
9. Start timer: `lapic_timer_start_periodic(250)`
10. Enter `ap_sched_loop()`

## Per-CPU Data

Each CPU has a `cpu_local_t` structure accessed via GS:

| Offset | Field | Description |
|---|---|---|
| 0 | `kernel_rsp` | Kernel stack pointer |
| 8 | `user_rsp` | User stack pointer |
| 16 | `cpu_id` | CPU identifier |
| 32 | `current` | Current process |
| 40 | `idle` | Idle process |

Maximum supported CPUs is defined by `MAX_CPUS`.

Last reviewed: 2026-07-22
