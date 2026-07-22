# Process Control

This document describes the process control system calls (syscalls) in the Kyronix kernel. It is the child of [Syscalls](index.md).

## Syscall Table

- 56 `clone` - Create a new process/thread
- 57 `fork` - Create a child process
- 58 `vfork` - Create a child process (treated as fork)
- 59 `execve` - Execute a program
- 60 `exit` - Terminate a process
- 61 `wait4` - Wait for a process to change state
- 231 `exit_group` - Terminate all threads in a process
- 247 `waitid` - Wait for a specific process state change

- 39 `getpid` - Get process id (PID)
- 110 `getppid` - Get parent PID
- 186 `gettid` - Get thread id (TID)
- 102 `getuid` - Get real user id (UID)
- 104 `getgid` - Get real group id (GID)
- 107 `geteuid` - Get effective UID
- 108 `getegid` - Get effective GID

- 105 `setuid` - Set real UID
- 106 `setgid` - Set real GID
- 109 `setpgid` - Set process group id (PGID)
- 112 `setsid` - Create a session and set session id (SID)
- 113 `setreuid` - Set real and effective UID
- 114 `setregid` - Set real and effective GID

- 115 `getgroups` - Get supplementary group list
- 116 `setgroups` - Set supplementary group list
- 117 `setresuid` - Set real, effective, and saved UID
- 118 `getresuid` - Get real, effective, and saved UID
- 119 `setresgid` - Set real, effective, and saved GID
- 120 `getresgid` - Get real, effective, and saved GID

- 121 `getpgid` - Get PGID
- 122 `setfsuid` - Set filesystem UID
- 123 `setfsgid` - Set filesystem GID
- 124 `getsid` - Get session ID

- 158 `arch_prctl` - Architecture-specific process register (ARCH_SET_FS, ARCH_SET_GS, ARCH_GET_FS, ARCH_GET_GS)

## clone Flags

The `clone` syscall accepts flags to control process/thread creation behavior:

- `CLONE_VM` - Share memory with parent process
- `CLONE_FILES` - Share file descriptor table with parent
- `CLONE_THREAD` - Create a thread (shared address space)
- `CLONE_SETTLS` - Set thread-local storage (TLS)
- `CLONE_PARENT_SETTID` - Store child TID at parent-provided address
- `CLONE_CHILD_CLEARTID` - Clear child TID at exit and wake waiters
- `CLONE_CHILD_SETTID` - Store child TID at child-provided address

## execve Details

The `execve` syscall performs the following operations:

1. Reads the shebang line (#!) if present and interprets the interpreter path
2. Loads the executable binary (Executable and Linkable Format (ELF))
3. For dynamically linked executables, sets up the dynamic linker via `PT_INTERP` segment at virtual address 0x7f0000000000
4. Sets Position Independent Executable (PIE) base address at 0x400000
5. Sets up the user stack with `argc`, `argv`, `envp`, and auxiliary vector (`auxv`)
6. Initializes Address Space Layout Randomization (ASLR) for the memory map bump allocator (`mmap_bump`)

## fork Details

The `fork` syscall performs the following operations:

1. Deep-copies the address space via `vmm_fork_user()`
2. Copies the file descriptor table
3. Creates a new kernel stack for the child process
4. The child process returns with `rax` set to 0

## clone Thread Support

The `clone` syscall with `CLONE_THREAD` flag creates a thread that shares:

- The address space (via `CLONE_VM`)
- The file descriptor table (via `CLONE_FILES`)

## wait4 Details

The `wait4` syscall supports the following features:

- Zombie reaping of terminated children
- Handling of `ptrace`-stopped tracees
- Job-stopped children
- Non-blocking operation via `WNOHANG` flag

## proc_do_exit

The `proc_do_exit` function performs the following cleanup operations:

1. Cleans up System V Shared Memory (SHM)
2. Unreferences the jail
3. Releases the file descriptor table
4. Reparents children to PID 1 (init process)
5. Delivers `SIGCHLD` signal to the parent process

Last reviewed: 2026-07-22
