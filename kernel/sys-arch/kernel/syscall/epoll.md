# Epoll

This document describes the epoll event polling interface in the Kyronix kernel. It is the child of [Syscalls](index.md).

## Syscall Table

- 213 `epoll_create` - Create an epoll instance (delegates to `epoll_create1(0)`)
- 232 `epoll_wait` - Wait for events on an epoll instance
- 233 `epoll_ctl` - Control an epoll instance
- 281 `epoll_pwait` - Wait for events with signal mask (delegates to `epoll_wait`)
- 291 `epoll_create1` - Create an epoll instance with flags

## Operations

The `epoll_ctl` (syscall 233) syscall modifies the set of file descriptors monitored by an epoll instance:

- `EPOLL_CTL_ADD` (1) - Add a file descriptor to the epoll interest list. Returns `EEXIST` if the file descriptor is already monitored. Returns `ENOMEM` if the watch limit is reached.
- `EPOLL_CTL_DEL` (2) - Remove a file descriptor from the epoll interest list. Returns `ENOENT` if the file descriptor is not found.
- `EPOLL_CTL_MOD` (3) - Modify the events and data associated with an existing file descriptor. Returns `ENOENT` if the file descriptor is not found.

## Events

| Event | Value | Description |
|---|---|---|
| `EPOLLIN` | 0x001 | Data is available for read |
| `EPOLLOUT` | 0x004 | Ready for write |
| `EPOLLERR` | 0x008 | Error condition |
| `EPOLLHUP` | 0x010 | Hang up |
| `EPOLLONESHOT` | 0x40000000 | One-shot: disarmed after first event delivery |

Events `EPOLLERR` and `EPOLLHUP` are reported regardless of whether they are in the interest set. The `EPOLLONESHOT` flag disables the watch after a single event; the watch must be re-armed via `EPOLL_CTL_MOD`.

## Implementation

Epoll state is managed by the `g_epolls` array with `EPOLL_SLOTS` (64) entries. Each epoll instance contains:

- `epfd` - The file descriptor handle (backed by an internal `/dev/null` file descriptor)
- `owner_space` - The VMM (Virtual Memory Manager) address space that owns the instance
- `w[EPOLL_MAXW]` - Array of `EPOLL_MAXW` (256) watch entries, each containing a file descriptor, events mask, and user data
- `nw` - Current number of active watches

Epoll instances are associated with the creating process's address space. An instance is only findable by processes sharing the same address space. Stale instances (where the backing file descriptor has been closed) are cleaned up during `epoll_create1`.

## Polling

The `epoll_wait` (syscall 232) and `epoll_pwait` (syscall 281) syscalls poll all watches in the epoll interest list:

1. For each watch, check `fd_valid()` to determine if the file descriptor is still open. If not, report `EPOLLERR | EPOLLHUP`.
2. If the watch requests `EPOLLIN`, check `fd_pollin()` for available read data.
3. If the watch requests `EPOLLOUT`, check `fd_pollout()` for write readiness.
4. Check `fd_pollhup()` unconditionally for hang-up conditions.
5. If `EPOLLONESHOT` is set on a triggered watch, disarm the `EPOLLIN` and `EPOLLOUT` bits.

The polling loop runs at 5-millisecond intervals (`p->wakeup_tick = g_ticks + 5`). If events are found or the timeout expires, the syscall returns immediately. If a signal is pending, the syscall returns `EINTR`.

The `timeout` parameter controls the maximum wait duration:

- `timeout == 0` - Non-blocking poll, return immediately
- `timeout > 0` - Block for up to `timeout` milliseconds
- `timeout < 0` - Block indefinitely until events are found or a signal arrives

The `epoll_create` (syscall 213) syscall delegates to `epoll_create1(0)`. The `epoll_pwait` (syscall 281) syscall delegates to `epoll_wait` without applying a signal mask.

Last reviewed: 2026-07-22
