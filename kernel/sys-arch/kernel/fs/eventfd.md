# eventfd

This document describes the eventfd implementation in the Kyronix kernel.

## Overview

eventfd provides a lightweight signaling mechanism between processes or threads. It is implemented as a file descriptor with an internal 64-bit counter.

## Structure

```c
typedef struct {
    volatile uint64_t counter;  // event counter
    uint32_t semaphore;         // semaphore for blocking
    void *waiter;               // waiting process
    spinlock_t lock;            // synchronization
} eventfd_state_t;
```

## Operations

| Operation | Description |
|-----------|-------------|
| `read()` | If counter > 0, returns the counter value and resets to 0. If counter = 0, blocks until counter > 0 (unless EFD_NONBLOCK). |
| `write()` | Adds the written 64-bit value to the counter. If `EFD_SEMAPHORE` is set, decrements by 1 instead. Wakes any blocked readers. |
| `close()` | Frees the eventfd state. |

## Flags

| Flag | Description |
|------|-------------|
| `EFD_SEMAPHORE` | Read decrements by 1 (semaphore mode) |
| `EFD_NONBLOCK` | Non-blocking I/O |

## Creation

`fd_eventfd(initval, flags)` creates a new eventfd file descriptor with the specified initial value and flags.

## Use Cases

- Event notification between threads
- Waiting for multiple file descriptors (via poll/select/epoll)
- Semaphore-like counting
