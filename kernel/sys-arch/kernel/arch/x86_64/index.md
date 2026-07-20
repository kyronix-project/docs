# x86-64 Architecture

This section describes the x86-64 architecture-specific subsystems in the Kyronix kernel.

## Components

- [CPU Primitives](cpu.md) -- port I/O, MSR access, control registers, SSE initialization
- [GDT](gdt.md) -- Global Descriptor Table and TSS setup
- [IDT](idt.md) -- Interrupt Descriptor Table and ISR dispatch
- [LAPIC](lapic.md) -- Local APIC timer and IPI delivery
- [PIT](pit.md) -- Programmable Interval Timer
- [Syscall Entry](syscall.md) -- SYSCALL/SYSRET MSR configuration and entry point

## Synchronization Primitives

- **Spinlock** (`spinlock.h`): CAS-based spinlock with `cpu_relax()` in the spin loop. Supports `spin_lock_irqsave` / `spin_unlock_irqrestore` for interrupt-safe locking. Also provides a Big Kernel Lock (BKL) with owner tracking and recursive depth.
- **Atomics** (`atomic.h`): Lock-prefixed x86 atomic operations (xchg, cmpxchg, fetch_add, inc, dec) and memory barriers (mfence, lfence, sfence).
- **Per-CPU data** (`percpu.h`): GS-segment-based per-CPU data structure (`cpu_local_t`) containing kernel stack pointer, user stack, CPU ID, LAPIC ID, current process, address space, file descriptor table, and reap thread.
