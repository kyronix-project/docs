# Protocols

This document describes the IPC and communication protocols used in Kyronix.

## Kernel-to-Userspace

### Syscall Interface

The primary kernel-userspace interface is the x86-64 `SYSCALL`/`SYSRET` mechanism. Arguments are passed in registers (rdi, rsi, rdx, r10, r8, r9) and the result is returned in rax.

### Signal Delivery

Signals are delivered by modifying the user stack with an `rt_sigframe_t` and redirecting execution to the signal handler.

## Userspace-to-Userspace

### Unix Domain Sockets

Used for IPC between servers and applications:

- `AF_UNIX`, `SOCK_STREAM` or `SOCK_DGRAM`
- Supports `SCM_RIGHTS` for file descriptor passing
- Supports `SCM_CREDENTIALS` for peer credential retrieval

### Pipes

Used for sequential data flow between processes:

- Unidirectional (read end, write end)
- Ring buffer implementation
- Supports poll/select/epoll

### Eventfd

Used for event notification:

- 64-bit counter
- Semaphore mode (`EFD_SEMAPHORE`)
- Integrates with poll/select/epoll

## Server Protocols

### mbus Protocol

The message bus uses bragi-serialized messages for:

- Device registration
- Device discovery
- Server capability queries

### POSIX Subsystem Protocol

The posix-subsystem communicates with the kernel via standard syscalls and with other servers via bragi-serialized IPC messages.
