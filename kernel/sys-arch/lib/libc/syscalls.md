# Syscall Wrappers

This document describes the syscall wrapper interface between libc and the Kyronix kernel.

## Overview

The kernel implements the Linux x86-64 syscall ABI. Musl libc provides thin assembly wrappers that invoke syscalls using the `syscall` instruction.

## Convention

The syscall convention follows Linux:

```c
long syscall(long number, long a1, long a2, long a3, long a4, long a5, long a6);
```

Assembly implementation:

```asm
syscall_wrapper:
    mov rax, rdi      ; syscall number
    mov rdi, rsi      ; arg1
    mov rsi, rdx      ; arg2
    mov rdx, rcx      ; arg3
    mov r10, r8       ; arg4 (rcx is clobbered by syscall)
    mov r8, r9        ; arg5
    syscall
    ret
```

## Syscall Numbers

The kernel uses the standard Linux x86-64 syscall numbers. A subset:

| NR | Name | NR | Name |
|----|------|----|------|
| 0 | read | 1 | write |
| 2 | open | 3 | close |
| 9 | mmap | 10 | mprotect |
| 11 | munmap | 12 | brk |
| 35 | nanosleep | 39 | getpid |
| 41 | socket | 56 | clone |
| 57 | fork | 59 | execve |
| 60 | exit | 61 | wait4 |
| 62 | kill | 202 | futex |
| 228 | clock_gettime | 231 | exit_group |

## Custom Syscalls

Kyronix adds custom syscalls for the jail subsystem (numbers 500-506). These are not part of the standard Linux ABI and require custom libc wrappers.

## Error Return

Syscalls return the result in `rax`. On error, `rax` contains a negative errno value (e.g., `-EINVAL = -22`). The libc wrapper converts this to setting `errno` and returning `-1`.
