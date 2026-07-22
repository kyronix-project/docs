# Syscalls Reference

This document provides a complete reference of all syscalls implemented in the Kyronix kernel. It is the child of [Reference](index.md).

## Overview

The kernel implements approximately 140 Linux x86_64 ABI-compatible syscalls plus custom jail syscalls.

## Syscall Table

| #  | Name             | #  | Name             |
|----|------------------|----|------------------|
| 0  | read             | 1  | write            |
| 2  | open             | 3  | close            |
| 4  | stat             | 5  | fstat            |
| 6  | lstat            | 7  | poll             |
| 8  | lseek            | 9  | mmap             |
| 10 | mprotect         | 11 | munmap           |
| 12 | brk              | 13 | rt_sigaction     |
| 14 | rt_sigprocmask   | 15 | rt_sigreturn     |
| 16 | ioctl            | 17 | pread64          |
| 18 | pwrite64         | 19 | readv            |
| 20 | writev           | 21 | access           |
| 22 | pipe             | 23 | select           |
| 25 | mremap           | 29 | shmget           |
| 30 | shmat            | 31 | shmctl           |
| 32 | dup              | 33 | dup2             |
| 34 | pause            | 35 | nanosleep        |
| 36 | getitimer        | 37 | alarm            |
| 38 | setitimer        | 39 | getpid           |
| 40 | sendfile         | 41 | socket           |
| 42 | connect          | 43 | accept           |
| 44 | sendto           | 45 | recvfrom         |
| 46 | sendmsg          | 47 | recvmsg          |
| 48 | shutdown         | 49 | bind             |
| 50 | listen           | 51 | getsockname      |
| 52 | getpeername      | 53 | socketpair       |
| 54 | setsockopt       | 55 | getsockopt       |
| 56 | clone            | 57 | fork             |
| 58 | vfork            | 59 | execve           |
| 60 | exit             | 61 | wait4            |
| 62 | kill             | 63 | uname            |
| 67 | shmdt            | 72 | fcntl            |
| 76 | truncate         | 77 | ftruncate        |
| 78 | getdents64       | 79 | getcwd           |
| 80 | chdir            | 82 | rename           |
| 83 | mkdir            | 84 | rmdir            |
| 86 | link             | 87 | unlink           |
| 88 | symlink          | 89 | readlink         |
| 90 | chmod            | 95 | umask            |
| 96 | gettimeofday     | 97 | getrlimit        |
| 101| ptrace           | 102| getuid           |
| 104| getgid           | 105| setuid           |
| 106| setgid           | 107| geteuid          |
| 108| getegid          | 109| setpgid          |
| 112| setsid           | 117| setresuid        |
| 119| setresgid        | 137| statfs           |
| 158| arch_prctl       | 169| reboot           |
| 186| gettid           | 201| time             |
| 202| futex            | 213| epoll_create     |
| 218| set_tid_address  | 228| clock_gettime    |
| 229| clock_getres     | 230| clock_nanosleep  |
| 231| exit_group       | 232| epoll_wait       |
| 233| epoll_ctl        | 234| tgkill           |
| 257| openat           | 262| newfstatat       |
| 263| unlinkat         | 270| pselect6         |
| 271| ppoll            | 280| openat2          |
| 283| timerfd_create   | 284| eventfd          |
| 288| accept4          | 290| eventfd2         |
| 291| epoll_create1    | 292| dup3             |
| 293| pipe2            | 295| preadv           |
| 296| pwritev          | 302| prlimit64        |
| 318| getrandom        | 319| memfd_create     |
| 326| copy_file_range  | 332| statx            |
| 334| close_range      |    |                  |

## Custom Jail Syscalls

| #   | Name              |
|-----|-------------------|
| 500 | jail_create       |
| 501 | jail_attach       |
| 502 | jail_get          |
| 503 | jail_list         |
| 504 | jail_remove       |
| 505 | jail_self         |
| 506 | jail_set_auto     |

Last reviewed: 2026-07-22
