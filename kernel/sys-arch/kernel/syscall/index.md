# Syscalls

This document describes the syscall interface of the Kyronix kernel. It is the child of [Kernel](sys-arch/kernel/index.md) and parent of syscall category documents.

## Overview

The Kyronix kernel implements a Linux x86_64 ABI-compatible syscall interface. User programs invoke syscalls via the `SYSCALL` instruction with the syscall number in RAX and arguments in RDI, RSI, RDX, R10, R8, R9.

## Syscall Entry

1. `swapgs` (switch to kernel GS base)
2. Save user RSP, load kernel RSP
3. Push all GPRs into `cpu_state_t` frame
4. Call `syscall_dispatch(frame)` in C
5. Restore registers, `sysretq` to user mode

## Categories

| Category | Document | Syscalls |
|---|---|---|
| File Operations | [file.md](file.md) | read, write, open, close, stat, lseek, dup, ioctl, ... |
| Process Control | [process.md](process.md) | fork, clone, execve, exit, wait4, ... |
| Memory Management | [memory.md](memory.md) | mmap, munmap, brk, mprotect, mremap, ... |
| Socket Operations | [socket.md](socket.md) | socket, connect, bind, listen, accept, sendto, recvfrom, ... |
| Timers | [timer.md](timer.md) | nanosleep, clock_gettime, alarm, setitimer, ... |
| Epoll | [epoll.md](epoll.md) | epoll_create1, epoll_ctl, epoll_wait, ... |
| Futex | [futex.md](futex.md) | futex (WAIT, WAKE, REQUEUE) |
| Ptrace | [ptrace.md](ptrace.md) | ptrace (TRACEME, PEEK, POKE, CONT, SINGLESTEP, ...) |

## Error Handling

Syscalls return errors via RAX. Negative values represent Linux errno constants (e.g., `-ENOENT`, `-EINVAL`).

## Signal Delivery

After every syscall, `signal_check()` is called to deliver pending signals. If a signal is delivered, the syscall may be restarted or interrupted based on signal handler configuration.

## Ptrace Integration

The syscall dispatcher checks `ptrace_syscall_trace` on every entry and exit. If set, the process is stopped with `SIGTRAP|0x80` for debugger observation.

Last reviewed: 2026-07-22
