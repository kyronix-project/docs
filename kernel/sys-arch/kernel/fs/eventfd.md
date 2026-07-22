# eventfd

This document describes the event file descriptor in the Kyronix kernel. It is the child of [Filesystems](sys-arch/kernel/fs/index.md).

## Source

`kernel/fs/eventfd.c`

## Overview

eventfd provides a lightweight signaling mechanism between processes or threads. It implements a 64-bit counter that can be read and written atomically.

## Syscalls

| Syscall | Number | Description |
|---|---|---|
| `eventfd` | 284 | Create eventfd with flags |
| `eventfd2` | 290 | Create eventfd with flags (O_CLOEXEC, O_NONBLOCK) |

## Operations

| Operation | Behavior |
|---|---|
| `read()` | Blocks if counter is 0; otherwise returns counter and resets to 0 |
| `write()` | Adds to the counter atomically |
| `poll()` | Returns readable when counter > 0, writable when counter < MAX |

## Implementation

eventfd instances are backed by a 64-bit atomic counter stored in the VFS node's data field. Reads and writes are performed atomically using spinlocks.

Last reviewed: 2026-07-22
