# Process Management

This section describes the process management subsystem of the Kyronix kernel.

## Components

- [Scheduler](scheduler.md) -- preemptive time-sliced scheduling
- [SMP](smp.md) -- Symmetric Multi-Processing support
- [Signal Handling](signal.md) -- POSIX signal delivery
- [Jail](jail.md) -- process sandboxing/isolation

## Process States

| State | Description |
|-------|-------------|
| `PROC_UNUSED` | Slot is free |
| `PROC_READY` | Process is ready to run |
| `PROC_RUNNING` | Process is currently executing |
| `PROC_WAITING` | Process is blocked (waiting for I/O, signal, etc.) |
| `PROC_ZOMBIE` | Process has exited, awaiting parent wait() |
| `PROC_DYING` | Process is being cleaned up |
| `PROC_STOPPED` | Process is stopped (job control, SIGSTOP) |

## Process Table

The kernel maintains a fixed-size process table with 64 slots (`PROC_MAX`). Each process is represented by a `proc_t` structure containing:

- Process IDs (pid, ppid, pgid)
- Address space (`vmm_space_t *space`)
- Kernel stack (16 pages, with guard page)
- File descriptor table (up to 1024 FDs)
- Signal state (pending signals, mask, actions)
- Credentials (uid, gid, euid, egid, etc.)
- Jail membership
- Ptrace state
- FPU state (512 bytes, FXSAVE format)

## Key Operations

| Function | Description |
|----------|-------------|
| `proc_init()` | Initialize the process table |
| `proc_alloc(ppid)` | Allocate a new process slot |
| `proc_find(pid)` | Find a process by PID |
| `proc_create_idle(cpu_id, entry)` | Create an idle process for a CPU |
| `proc_do_exit(code)` | Terminate the current process |
| `proc_defer_thread_reap(p)` | Defer cleanup of a dead thread |

## Reference Counting

Processes use atomic reference counting (`proc_ref` / `proc_unref`). When the reference count drops to 0, the kernel stack is freed and the slot is released.

## Process Limits

| Resource | Limit |
|----------|-------|
| Max processes | 64 |
| Kernel stack | 16 pages (64 KiB) per process |
| Max file descriptors | 1024 per process |
| Max VMA regions | 2048 per address space |
