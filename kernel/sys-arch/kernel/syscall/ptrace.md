# Ptrace

This document describes the process tracing (ptrace) interface in the Kyronix kernel. It is the child of [Syscalls](index.md).

## Syscall Table

- 101 `ptrace` - Process tracing and debugging

## Operations

The `ptrace` syscall supports the following request codes:

### PTRACE_TRACEME (0)

Sets the calling process's `tracer_pid` to its parent PID. The process becomes traceable by its parent. No arguments are used.

### PTRACE_ATTACH (16)

Attaches the calling process as a tracer to a target process:

1. Look up the target process by PID via `proc_find()`.
2. Verify the target is not already being traced (`tracer_pid == 0`). If already traced, return `EPERM`.
3. Set `t->tracer_pid = self->pid`.
4. Send `SIGSTOP` to the target process.
5. Return 0.

### PTRACE_PEEKTEXT / PTRACE_PEEKDATA (1, 2)

Reads 8 bytes from the target process's address space at address `addr`:

1. Switch to the target's address space via `vmm_switch()`.
2. Validate the user pointer with `uptr_ok()`.
3. Copy 8 bytes from the target address to a kernel buffer.
4. Switch back to the caller's address space.
5. Write the result to the `data` pointer in the caller's address space.

Both request codes behave identically.

### PTRACE_POKETEXT / PTRACE_POKEDATA (4, 5)

Writes 8 bytes to the target process's address space at address `addr`:

1. Switch to the target's address space via `vmm_switch()`.
2. Validate the user pointer with `uptr_ok_w()`.
3. Copy 8 bytes from the `data` argument to the target address.
4. Switch back to the caller's address space.

Both request codes behave identically.

### PTRACE_GETREGS (12)

Reads the full register state of the target process into a `ptrace_user_regs` structure:

1. Call `ptrace_fill_regs()` to populate the structure from the target's current frame.
2. Copy the structure to the caller's `data` pointer.

### PTRACE_SETREGS (13)

Writes a `ptrace_user_regs` structure to the target process's register state:

1. Copy the `ptrace_user_regs` structure from the caller's `data` pointer.
2. Call `ptrace_store_regs()` to apply the register values to the target's current frame.

### PTRACE_CONT (7)

Resumes the target process's execution without syscall tracing or single-stepping:

1. Verify the target is in a stopped state (`ptrace_stopped != 0`). Otherwise, return `ESRCH`.
2. If a signal number is provided in `data`, inject it as a pending signal.
3. Clear `ptrace_stopped` and `ptrace_reported`.
4. Transition the target from `PROC_WAITING` to `PROC_READY`.

### PTRACE_SYSCALL (24)

Resumes the target process's execution with syscall tracing enabled:

1. Same as `PTRACE_CONT`, but sets `ptrace_syscall_trace = 1`.
2. The syscall dispatcher checks `ptrace_syscall_trace` on entry and exit, stopping the process with `SIGTRAP|0x80` at each syscall boundary.

### PTRACE_SINGLESTEP (9)

Resumes the target process's execution with single-stepping enabled:

1. Same as `PTRACE_CONT`, but sets `ptrace_step = 1`.
2. The target executes one instruction before being stopped again.

### PTRACE_KILL (8)

Injects `SIGKILL` into the target process:

1. Set the `SIGKILL` bit in the target's `pending_sigs`.
2. Clear `ptrace_stopped` and `ptrace_reported`.
3. Transition the target from `PROC_WAITING` to `PROC_READY`.

### PTRACE_DETACH (17)

Detaches the tracer from the target process:

1. Clear `tracer_pid` and `ptrace_syscall_trace`.
2. If the target is stopped, clear `ptrace_stopped` and transition it to `PROC_READY`.

### PTRACE_SETOPTIONS (0x4200)

Accepted as a no-op. Returns 0.

## Register State

The `ptrace_user_regs` structure contains the full x86_64 general-purpose register state:

```
r15, r14, r13, r12, rbp, rbx, r11, r10, r9, r8,
rax, rcx, rdx, rsi, rdi, orig_rax, rip, cs, eflags,
rsp, ss, fs_base, gs_base, ds, es, fs, gs
```

Segment registers are set to kernel constants: `cs` = `GDT_USER_CODE_SEL`, `ss`/`ds`/`es`/`fs`/`gs` = `GDT_USER_DATA_SEL`. The `fs_base` is read from the target process's `fs_base` field.

## Frame Kinds

The `ptrace_frame_kind` field determines how register values are extracted and stored:

- **Frame kind 1** (`syscall_frame_t`) - Syscall entry frame. `RIP` is derived from `rcx` (the syscall return address). `RFLAGS` is derived from `r11`. This frame is used when the process is stopped at a syscall entry/exit via `SIGTRAP|0x80`.
- **Frame kind 2** (`cpu_state_t`) - Full interrupt frame. `RIP` and `RFLAGS` are read directly from the interrupt frame. This frame is used when the process is stopped via `#BP` (breakpoint) or `#DB` (debug) exceptions.

## Permission Model

Only the tracer process (identified by `t->tracer_pid == self->pid`) may operate on a tracee. All operations except `PTRACE_TRACEME` and `PTRACE_ATTACH` verify the caller is the tracer. If the caller is not the tracer, the syscall returns `ESRCH`.

## Exception Integration

The kernel's Interrupt Descriptor Table (IDT) checks `tracer_pid` on `#BP` (vector 3) and `#DB` (vector 1) exceptions:

1. On `#BP`, the instruction pointer is decremented by 1 (past the `int3` opcode).
2. The process's `ptrace_orig_rax` is saved.
3. `proc_ptrace_stop()` is called with frame kind 2, delivering `SIGTRAP` to stop the process.

## Syscall Dispatcher Integration

The syscall dispatcher checks `ptrace_syscall_trace` on every syscall entry and exit:

1. On entry, if `ptrace_syscall_trace` is set and the syscall number is not 101 (`ptrace` itself), the process is stopped with `SIGTRAP|0x80` and frame kind 1.
2. On exit, after storing the return value, the process is stopped again with `SIGTRAP|0x80` and frame kind 1.
3. The `ptrace_in_syscall` flag is set on entry and cleared on exit to distinguish entry stops from exit stops.

Last reviewed: 2026-07-22
