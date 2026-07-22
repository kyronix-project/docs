# Pipe

This document describes the pipe implementation in the Kyronix kernel. It is the child of [Filesystems](sys-arch/kernel/fs/index.md).

## Source

`kernel/fs/pipe.c`, `kernel/fs/fdpipe.c`

## Overview

Pipes provide unidirectional inter-process communication. The Kyronix kernel implements both anonymous pipes (`pipe`/`pipe2`) and named pipes (FIFOs).

## Syscalls

| Syscall | Number | Description |
|---|---|---|
| `pipe` | 22 | Create anonymous pipe |
| `pipe2` | 293 | Create pipe with flags (O_CLOEXEC, O_NONBLOCK) |

## Data Structure

```c
typedef struct {
    uint8_t *buf;        // Ring buffer
    size_t   size;       // Buffer capacity
    size_t   read_pos;   // Read position
    size_t   write_pos;  // Write position
    size_t   count;      // Bytes available
    // Synchronization primitives for blocking read/write
} pipe_t;
```

## Operations

| Operation | Behavior |
|---|---|
| `read()` | Blocks if empty; reads up to requested bytes |
| `write()` | Blocks if full; writes up to requested bytes |
| `close()` | Signals EOF to readers/writers |

## Ancillary Data (SCM_RIGHTS)

Pipes support file descriptor passing via `SCM_RIGHTS` ancillary data on Unix domain sockets. This enables processes to share file descriptors across address spaces.

## Blocking

Read and write operations block the calling process when the pipe is empty (read) or full (write). Blocked processes are moved to `PROC_WAITING` state and woken when data becomes available.

Last reviewed: 2026-07-22
