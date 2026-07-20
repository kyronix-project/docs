# Epoll

This document describes the epoll subsystem in the Kyronix kernel.

## Syscalls

| Syscall | Number | Description |
|---------|--------|-------------|
| `epoll_create1` | 291 | Create an epoll instance |
| `epoll_ctl` | 233 | Add/modify/remove file descriptors |
| `epoll_wait` | 232 | Wait for events |

## Overview

epoll provides efficient I/O event notification for monitoring multiple file descriptors. It is similar to Linux epoll.

## epoll_create1

`epoll_create1(flags)` creates a new epoll instance:

- `EPOLL_CLOEXEC`: Set close-on-exec on the epoll FD
- Returns a file descriptor referring to the epoll instance

## epoll_ctl

`epoll_ctl(epfd, op, fd, event)` controls which file descriptors are monitored:

| Operation | Description |
|-----------|-------------|
| `EPOLL_CTL_ADD` | Add a file descriptor |
| `EPOLL_CTL_MOD` | Modify events for a file descriptor |
| `EPOLL_CTL_DEL` | Remove a file descriptor |

### Event Types

| Event | Description |
|-------|-------------|
| `EPOLLIN` | Data available for reading |
| `EPOLLOUT` | Can write without blocking |
| `EPOLLHUP` | Hang up on connection |
| `EPOLLRDHUP` | Peer closed connection |
| `EPOLLERR` | Error condition |

### Flags

| Flag | Description |
|------|-------------|
| `EPOLLONESHOT` | Auto-disarm after one event |

## epoll_wait

`epoll_wait(epfd, events, maxevents, timeout)` waits for events:

1. Poll all monitored file descriptors
2. For each FD with a pending event, add it to the result set
3. If no events and timeout > 0, sleep for up to 5ms and retry
4. Return the number of ready file descriptors

## Implementation

- Slot-based design: 64 epoll instances, 256 watches each
- Owner-space tracking for cleanup when the owning process dies
- EPOLLONESHOT support (auto-disarm after event delivery)
- Polling via `fd_pollin` / `fd_pollout` / `fd_pollhup` with 5ms sleep intervals
- EINTR returned on signal delivery
