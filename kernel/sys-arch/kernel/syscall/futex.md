# Futex

This document describes the futex (Fast Userspace muTEX) implementation in the Kyronix kernel.

## Syscalls

| Syscall | Number | Description |
|---------|--------|-------------|
| `futex` | 202 | Fast userspace mutex |

## Operations

| Operation | Description |
|-----------|-------------|
| `FUTEX_WAIT` | Block if `*uaddr == val` |
| `FUTEX_WAKE` | Wake up to `val` waiters on `uaddr` |
| `FUTEX_REQUEUE` | Move waiters from one futex to another |
| `FUTEX_CMP_REQUEUE` | Like REQUEUE, but only if `*uaddr == val` |

## FUTEX_WAIT

`futex(uaddr, FUTEX_WAIT, val, timeout)`:

1. Verify that `*uaddr == val` (atomic read)
2. If not equal, return `EWOULDBLOCK`
3. Block the current process on the futex address
4. Wake on signal delivery or timeout

## FUTEX_WAKE

`futex(uaddr, FUTEX_WAKE, val)`:

1. Wake up to `val` processes waiting on `uaddr`
2. Only wakes processes in the same jail (jail-scoped)
3. Returns the number of processes woken

## FUTEX_REQUEUE

`futex(uaddr, FUTEX_REQUEUE, val, uaddr2)`:

1. Wake up to `val` waiters on `uaddr`
2. Move remaining waiters to `uaddr2`

## FUTEX_CMP_REQUEUE

Same as REQUEUE, but checks `*uaddr == val` before proceeding.

## Thread Cleanup

`cleartid_wake` is a special operation used for thread cleanup:

- Called when a thread exits
- Wakes all futex waiters on the clear-tid address
- Used with `CLONE_CHILD_CLEARTID`

## Jail Scoping

Futex wake operations are scoped to the current jail. Processes in different jails cannot wake each other's futexes, providing isolation for synchronization primitives.
