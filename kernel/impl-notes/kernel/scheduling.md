# Scheduling

This document describes the scheduling implementation in the Kyronix kernel. It is the child of [Kernel Implementation Notes](index.md).

The Kyronix kernel uses a per-CPU round-robin scheduler with lock-free bitmap-based ready-queue selection and hardware-assisted FPU context switching.

## Scheduling Algorithm

### Ready Queue and Bitmap

- Each CPU maintains a 64-bit ready bitmask (`g_ready_mask`) representing the priority levels with at least one runnable thread.
- Next-thread selection uses `__builtin_ctzll` (Count Trailing Zeros, Long Long) on `g_ready_mask` to locate the lowest-numbered set bit in O(1) time.
- Each priority level maps to a linked list of threads in the READY state.

### Lock-Free Claim

- `sched_claim_next` transitions a thread from READY to RUNNING using a Compare-And-Swap (CAS) operation, eliminating lock contention on the common path.
- The CAS atomically marks the thread as RUNNING before any scheduler state is visible to other processors.

### Round-Robin Fairness

- Per-CPU arrays (`g_last_scheduled`) track the last scheduled thread per priority level to enforce round-robin fairness across threads of equal priority.

## Context Switch

The context switch performs the following operations in sequence:

1. Save callee-saved general-purpose registers from the outgoing thread.
2. Execute `fxsave64` to save the floating-point / SIMD state of the outgoing thread.
3. Restore callee-saved general-purpose registers for the incoming thread.
4. Execute `fxrstor64` to restore the floating-point / SIMD state of the incoming thread.
5. Write the FS base MSR (Model-Specific Register) for the incoming thread's Thread-Local Storage (TLS).
6. Load CR3 (Control Register 3) to switch to the incoming thread's address space.

## Preemption

- Preemption is triggered on Programmable Interval Timer (PIT) IRQ 0 and Local APIC timer interrupt (vector 224).
- The timer interrupt handler invokes the scheduler to perform a context switch if a higher-priority or equal-priority thread is runnable.

## Application Processor Idle Loop

Application Processors (APs) execute the following idle loop:

1. Call `sched_claim_next` to attempt to acquire a runnable thread.
2. If a thread is claimed, call `sched_switch` to context-switch into it.
3. If no thread is available, execute `hlt` (Halt) until the next interrupt.

## Process States and Transitions

Threads transition through the following states:

```
UNUSED -> READY -> RUNNING -> READY
                       |-> WAITING
                       |-> ZOMBIE -> DYING
                       |-> STOPPED
```

- `UNUSED` — Thread slot is unallocated.
- `READY` — Thread is runnable and waiting for CPU time.
- `RUNNING` — Thread is executing on a CPU.
- `WAITING` — Thread is blocked on an event (e.g., I/O, sleep).
- `ZOMBIE` — Thread has exited but its resources have not yet been reclaimed.
- `DYING` — Thread is in the final stage of resource teardown.
- `STOPPED` — Thread has been stopped (e.g., via signal).

## Deferred Reaping

- `proc_defer_thread_reap` stores a zombie thread in a pending list for deferred cleanup.
- The next call to `proc_reap_pending` processes the deferred list and reclaims thread resources.
- Deferred reaping avoids performing memory deallocation in the interrupt context where the thread transitions to ZOMBIE.

Last reviewed: 2026-07-22
