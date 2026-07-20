# System Calls

This section describes the Linux-compatible syscall interface of the Kyronix kernel.

## Overview

Kyronix implements over 150 Linux x86-64 syscalls, enabling execution of unmodified Linux binaries compiled with musl libc. The syscall convention follows the Linux x86-64 ABI.

## Syscall Convention

| Register | Purpose |
|----------|---------|
| `rax` | Syscall number |
| `rdi` | arg1 |
| `rsi` | arg2 |
| `rdx` | arg3 |
| `r10` | arg4 |
| `r8` | arg5 |
| `r9` | arg6 |
| `rax` (return) | Return value or negative errno |

## Categories

- [File Operations](file.md) -- read, write, open, close, stat, ioctl, etc.
- [Process Control](process.md) -- fork, clone, exec, exit, wait, etc.
- [Memory Management](memory.md) -- mmap, munmap, mprotect, brk, etc.
- [Socket Operations](socket.md) -- socket, connect, bind, sendmsg, recvmsg, etc.
- [Timer Operations](timer.md) -- nanosleep, clock_gettime, alarm, etc.
- [Epoll](epoll.md) -- epoll_create, epoll_ctl, epoll_wait
- [Futex](futex.md) -- FUTEX_WAIT, FUTEX_WAKE, FUTEX_REQUEUE
- [Ptrace](ptrace.md) -- process tracing and debugging

## Dispatch

The syscall dispatcher (`syscall.c`) reads the syscall number from `rax` and dispatches via a `switch` statement to the appropriate handler function.

## Jail Syscalls

Kyronix adds custom syscalls for the jail subsystem:

| Syscall Number | Name | Description |
|----------------|------|-------------|
| 500 | `jail_create` | Create a new jail |
| 501 | `jail_attach` | Attach process to a jail |
| 502 | `jail_get` | Get jail information |
| 503 | `jail_list` | List all jails |
| 504 | `jail_remove` | Remove a jail |
| 505 | `jail_self` | Get current jail ID |
| 506 | `jail_set_auto` | Enable/disable auto-isolation |

## Error Handling

Syscalls return a negative errno value on failure. Common errnos:

| Errno | Value | Description |
|-------|-------|-------------|
| EPERM | 1 | Operation not permitted |
| ENOENT | 2 | No such file or directory |
| ESRCH | 3 | No such process |
| EINTR | 4 | Interrupted system call |
| EACCES | 13 | Permission denied |
| EEXIST | 17 | File exists |
| EINVAL | 22 | Invalid argument |
| ENOMEM | 12 | Out of memory |
| ENOSPC | 28 | No space left on device |
