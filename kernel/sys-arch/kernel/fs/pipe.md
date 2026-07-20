# Pipe

This document describes the pipe and FIFO implementation in the Kyronix kernel.

## Overview

Pipes provide unidirectional inter-process communication (IPC). Data written to the write end is read from the read end in FIFO order.

## Structure

```c
typedef struct {
    uint8_t *buf;       // ring buffer
    uint64_t size;      // buffer capacity
    uint64_t read_pos;  // read position
    uint64_t write_pos; // write position
    uint64_t count;     // bytes currently in buffer
    // synchronization
    spinlock_t lock;
    uint32_t semaphore;
    void *read_waiter;
    void *write_waiter;
} pipe_t;
```

## Operations

| Operation | Description |
|-----------|-------------|
| `write()` | Write data to the pipe. If the pipe is full, blocks until space is available (unless O_NONBLOCK). |
| `read()` | Read data from the pipe. If the pipe is empty, blocks until data is available (unless O_NONBLOCK). |
| `close()` | Close one end of the pipe. When all write ends are closed, readers get EOF. When all read ends are closed, writers get SIGPIPE. |

## Creation

- `fd_pipe(pipefd[2])` -- create an unnamed pipe pair
- Named pipes (FIFOs) are supported via the filesystem

## Pipe Pair

A pipe consists of two `vfs_file_t` entries sharing a single `pipe_t`:

- `pipefd[0]` -- read end
- `pipefd[1]` -- write end

## Poll Support

Pipes support `poll`/`select`/`epoll`:

- `fd_pollin()` -- true when data is available or write end is closed
- `fd_pollout()` -- true when space is available or read end is closed
- `fd_pollhup()` -- true when write end is closed

## Buffer Size

The default pipe buffer size is implementation-defined. The buffer is allocated from the kernel heap.
