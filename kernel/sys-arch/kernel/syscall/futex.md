# Futex

This document describes the futex (Fast Userspace Mutex) implementation in the Kyronix kernel. It is the child of [Syscalls](index.md).

## Syscall Table

- 202 `futex` - Fast Userspace Mutex operations

## Operations

The `futex` syscall supports four operation commands, selected by the `op` argument. The `FUTEX_PRIVATE_FLAG` (128) and `FUTEX_CLOCK_REALTIME` (256) flags may be OR'd with the operation but are currently ignored.

### FUTEX_WAIT (0)

Atomically checks that the value at `uaddr` equals `val`, then blocks the calling process until woken:

1. Verify `*uaddr == val`. If not, return `EAGAIN`.
2. Find a free slot in `g_futex_tab`. If no slot is available, return `ENOMEM`.
3. Register the current process and `uaddr` in the slot.
4. If a timeout is provided (a `timespec` pointer), compute the deadline in milliseconds: `deadline = g_ticks + (sec * 1000 + nsec / 1000000)`.
5. Block in a loop until the process is woken, the deadline expires, or a signal is delivered.
6. On timeout, return `ETIMEDOUT`. Otherwise, return 0.

### FUTEX_WAKE (1)

Wakes up to `val` processes waiting on `uaddr`:

1. Scan `g_futex_tab` for entries matching `uaddr`.
2. For each matching entry, verify the waiting process is in the same jail as the caller (`g_futex_tab[i].proc->jail_id == self->jail_id`).
3. Transition the waiting process from `PROC_WAITING` to `PROC_READY` via `proc_set_ready()`.
4. Return the count of processes woken.

### FUTEX_REQUEUE (3)

Moves waiters from `uaddr` to `uaddr2`:

1. Wake up to `val` waiters on `uaddr`.
2. Requeue up to the second argument (passed in the timeout slot) waiters from `uaddr` to `uaddr2` by updating their `uaddr` field.
3. Cross-jail checks apply: only processes in the same jail as the caller are affected.

### FUTEX_CMP_REQUEUE (4)

Same as `FUTEX_REQUEUE`, but first verifies that `*uaddr == val3`. Returns `EAGAIN` if the comparison fails.

## Data Structure

Futex state is stored in the global `g_futex_tab` array with `FUTEX_MAX_WAITERS` (`PROC_MAX`) entries. Each entry contains:

- `uaddr` - The userspace address being waited on
- `proc` - Pointer to the waiting process (NULL if the slot is free)

The table is protected by `g_futex_lock`, a spinlock acquired on all read and write operations. Waiter registration and wakeup use `__sync_bool_compare_and_swap` for atomic state transitions.

## CLONE_CHILD_CLEARTID Integration

When a thread created with `CLONE_CHILD_CLEARTID` (flag 0x00200000) exits:

1. The kernel writes zero to the `cleartid_addr` stored in the thread's process structure.
2. The kernel calls `cleartid_wake(cleartid_addr)`.
3. `cleartid_wake` scans `g_futex_tab` for all entries whose `uaddr` matches `cleartid_addr`.
4. Each matching process is transitioned from `PROC_WAITING` to `PROC_READY`.

This mechanism enables pthread library implementations to use `CLONE_CHILD_CLEARTID` with a futex address, ensuring that joiners blocked on `futex(CLEARTID)` are woken when the thread exits.

Last reviewed: 2026-07-22
