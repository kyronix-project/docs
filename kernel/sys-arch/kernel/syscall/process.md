# Process Control

This document describes the process control syscalls in the Kyronix kernel.

## Syscalls

| Syscall | Number | Description |
|---------|--------|-------------|
| `clone` | 56 | Create a thread or process |
| `fork` | 57 | Fork a new process |
| `vfork` | 58 | Fork (shared address space) |
| `execve` | 59 | Execute a program |
| `exit` | 60 | Exit the current process |
| `wait4` | 61 | Wait for process state change |
| `exit_group` | 231 | Exit all threads in the process |
| `set_tid_address` | 96 | Set TID address for cleartid |
| `arch_prctl` | 158 | Architecture-specific process control |

## fork

`fork()` creates a new process by duplicating the calling process:

1. Allocate a new process slot
2. Create a new address space via `vmm_space_new()`
3. Copy the entire user-half page table via `vmm_fork_user()`
4. Copy the file descriptor table
5. Copy signal state, credentials, jail ID
6. Set the child's state to `PROC_READY`

> **Note:** This is a full-copy fork (not copy-on-write). All user pages are duplicated immediately.

## clone

`clone()` creates a new thread with configurable sharing:

| Flag | Description |
|------|-------------|
| `CLONE_VM` | Share address space (thread) |
| `CLONE_FILES` | Share file descriptor table |
| `CLONE_THREAD` | Create a thread (same thread group) |
| `CLONE_SETTLS` | Set thread-local storage |
| `CLONE_CHILD_CLEARTID` | Clear TID on child exit |
| `CLONE_PARENT` | Share parent |

## execve

`execve(path, argv, envp)` replaces the current process image:

1. Validate the path and arguments
2. Create a new address space
3. Load the ELF binary (PT_LOAD segments)
4. Handle shebang (`#!`) interpreter scripts
5. Set up the user stack with arguments and environment
6. Set the instruction pointer to the ELF entry point

### ELF Loading

The loader supports:

- 64-bit ELF executables
- `PT_LOAD` segments (loaded into memory)
- Position-independent executables (PIE)
- Dynamic linker loading at `0x7f0000000000`

### Shebang Support

If the executable starts with `#!`, the kernel:

1. Reads the interpreter path from the first line
2. Loads the interpreter (e.g., `/lib/ld-musl-x86_64.so.1`)
3. Passes the original executable as an argument to the interpreter

## exit

`exit(code)` terminates the current process:

1. Set state to `PROC_DYING`
2. Close all file descriptors
3. Release address space
4. Notify parent with `SIGCHLD`
5. Reparent children to PID 1
6. Release jail reference
7. Enter zombie state for parent to reap

## wait4

`wait4(pid, status, options)` waits for a child process state change:

- `WUNTRACED` -- report stopped processes
- `WCONTINUED` -- report continued processes
- Returns PID of the changed process and status information

## Job Control

Stopped processes (SIGSTOP/SIGTSTP) enter `PROC_STOPPED` state and notify their parent via `SIGCHLD`. The parent can:

- `wait4(WUNTRACED)` to detect the stop
- `kill(SIGCONT)` to resume the process
