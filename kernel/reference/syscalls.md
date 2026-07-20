# Syscalls Reference

This document provides a complete reference of all syscalls implemented in the Kyronix kernel.

## Syscall Numbers

| NR | Name | NR | Name | NR | Name |
|----|------|----|------|----|------|
| 0 | read | 1 | write | 2 | open |
| 3 | close | 4 | stat | 5 | fstat |
| 6 | lstat | 7 | poll | 8 | lseek |
| 9 | mmap | 10 | mprotect | 11 | munmap |
| 12 | brk | 13 | rt_sigaction | 14 | rt_sigprocmask |
| 15 | rt_sigreturn | 17 | pread64 | 18 | pwrite64 |
| 19 | readv | 20 | writev | 21 | access |
| 22 | pipe | 23 | select | 25 | mremap |
| 28 | madvise | 32 | dup | 33 | dup2 |
| 35 | nanosleep | 36 | getitimer | 37 | alarm |
| 38 | setitimer | 39 | getpid | 41 | socket |
| 42 | connect | 43 | accept | 44 | sendto |
| 45 | recvfrom | 46 | sendmsg | 47 | recvmsg |
| 48 | shutdown | 49 | bind | 50 | listen |
| 51 | getsockname | 52 | getpeername | 53 | socketpair |
| 54 | setsockopt | 55 | getsockopt | 56 | clone |
| 57 | fork | 58 | vfork | 59 | execve |
| 60 | exit | 61 | wait4 | 62 | kill |
| 63 | uname | 72 | fcntl | 79 | getcwd |
| 80 | chdir | 82 | rename | 83 | mkdir |
| 84 | rmdir | 85 | creat | 87 | unlink |
| 89 | readlink | 93 | truncate | 94 | ftruncate |
| 96 | gettimeofday | 98 | getrlimit | 99 | sysinfo |
| 100 | times | 101 | ptrace | 158 | arch_prctl |
| 202 | futex | 217 | getdents64 | 228 | clock_gettime |
| 229 | clock_getres | 230 | clock_nanosleep | 231 | exit_group |
| 232 | epoll_wait | 233 | epoll_ctl | 257 | openat |
| 258 | mkdirat | 262 | newfstatat | 263 | unlinkat |
| 264 | renameat | 265 | readlinkat | 268 | fchmodat |
| 269 | accessat | 291 | epoll_create1 | 292 | dup3 |
| 293 | pipe2 | 302 | prlimit64 | 318 | getrandom |
| 332 | statx | 334 | close_range | 500 | jail_create |
| 501 | jail_attach | 502 | jail_get | 503 | jail_list |
| 504 | jail_remove | 505 | jail_self | 506 | jail_set_auto |

## Return Values

- On success: non-negative result in `rax`
- On error: negative errno in `rax` (e.g., `-EINVAL = -22`)
- The libc wrapper converts this to `errno` assignment and `-1` return
