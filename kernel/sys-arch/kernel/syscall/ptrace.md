# Ptrace

This document describes the ptrace (process trace) implementation in the Kyronix kernel.

## Syscalls

| Syscall | Number | Description |
|---------|--------|-------------|
| `ptrace` | 101 | Process tracing and debugging |

## Operations

| Request | Description |
|---------|-------------|
| `PTRACE_TRACEME` | Request to be traced by parent |
| `PTRACE_PEEKTEXT` | Read a word from tracee's memory |
| `PTRACE_PEEKDATA` | Read a word from tracee's memory |
| `PTRACE_POKETEXT` | Write a word to tracee's memory |
| `PTRACE_POKEDATA` | Write a word to tracee's memory |
| `PTRACE_GETREGS` | Get register state |
| `PTRACE_SETREGS` | Set register state |
| `PTRACE_CONT` | Continue execution |
| `PTRACE_SINGLESTEP` | Execute one instruction |
| `PTRACE_SYSCALL` | Stop at syscall entry/exit |
| `PTRACE_KILL` | Kill the tracee |
| `PTRACE_ATTACH` | Attach to a process |
| `PTRACE_DETACH` | Detach from a process |

## Trace Mechanism

When a process is traced:

1. The tracee stores `tracer_pid` (the tracer's PID)
2. On signals (except SIGKILL), the tracee enters ptrace-stop
3. The tracer receives `SIGCHLD` and can inspect/modify the tracee
4. The tracer uses `wait4` to detect ptrace stops
5. The tracer uses `PTRACE_CONT`, `PTRACE_SINGLESTEP`, or `PTRACE_SYSCALL` to resume

## Register Access

`PTRACE_GETREGS` / `PTRACE_SETREGS` provide access to the full x86-64 register set, including:

- General purpose registers (rax-r15)
- Segment registers (cs, ds, es, fs, gs, ss)
- Instruction pointer (rip)
- Flags (rflags)
- FS base (for thread-local storage)

## Frame Kinds

The ptrace implementation supports two frame types:

| Kind | Description |
|------|-------------|
| 1 | `syscall_frame_t` -- entered via SYSCALL instruction |
| 2 | `cpu_state_t` -- entered via interrupt/exception |

## Memory Access

`PTRACE_PEEKTEXT` / `PTRACE_PEEKDATA` read a word (8 bytes) from the tracee's address space:

1. Translate the tracee's virtual address to physical
2. Read the value from the physical address

`PTRACE_POKETEXT` / `PTRACE_POKEDATA` write a word similarly.
