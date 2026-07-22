# Syscall Entry

This document describes the syscall entry mechanism in the Kyronix kernel. It is the child of [x86-64 Architecture](sys-arch/kernel/arch/x86_64/index.md).

## Sources

`kernel/arch/x86_64/syscall_setup.c`, `kernel/arch/x86_64/syscall_entry.S`

## Overview

The Kyronix kernel uses the AMD64 `SYSCALL`/`SYSRET` mechanism for system calls. This provides a fast, non-interrupt-based transition between user mode (ring 3) and kernel mode (ring 0).

## MSR Configuration

| MSR | Value | Purpose |
|---|---|---|
| `MSR_EFER` (0xC0000080) | SCE bit set | Enable SYSCALL support |
| `MSR_STAR` (0xC0000081) | Segments encoded | User CS/SS = 0x20/0x28, Kernel CS/SS = 0x08/0x18 |
| `MSR_LSTAR` (0xC0000082) | `syscall_entry` | SYSCALL entry point address |
| `MSR_SFMASK` (0xC0000084) | IF, TF, DF, AC | RFLAGS bits cleared on SYSCALL |

## SYSCALL Entry Sequence

The `syscall_entry` label in `syscall_entry.S`:

1. `swapgs` -- switch GS to kernel per-CPU data
2. Save user RSP to `gs:CPU_USER_RSP`
3. Load kernel RSP from `gs:CPU_KERNEL_RSP`
4. Push all 15 GPRs (forming a `cpu_state_t`-compatible frame)
5. Move RSP to RDI (first argument = pointer to register frame)
6. `call syscall_dispatch` (C function)
7. Pop all GPRs in reverse
8. Restore user RSP from `gs:CPU_USER_RSP`
9. `swapgs` -- switch GS back to user per-CPU data
10. `sysretq` -- return to ring 3

## Userspace Trampolines

| Function | Description |
|---|---|
| `enter_userspace(rip, rsp, rflags)` | First entry to ring 3 (no swapgs) |
| `enter_userspace_exec(rip, rsp, rflags)` | Entry after exec (includes swapgs) |

Both functions zero all GPRs except RIP, RSP, and RFLAGS, then execute `sysretq`.

## SSE/FPU Setup

SSE is enabled before SYSCALL configuration:

1. Clear CR0.EM (bit 2) -- disable emulation
2. Set CR0.MP (bit 1) -- monitor coprocessor
3. Set CR4.OSFXSR (bit 9) -- enable FXSAVE/FXRSTOR
4. Set CR4.OSXMMEXCPT (bit 10) -- enable unmasked SSE exceptions
5. `fninit` + `ldmxcsr 0x1F80` -- initialize x87 and SSE defaults

## Per-CPU Local Data

`cpu_local_t` is a 64-byte aligned structure accessed via GS:

```c
typedef struct {
    uint64_t kernel_rsp;    // offset 0
    uint64_t user_rsp;      // offset 8
    uint32_t cpu_id;        // offset 16
    uint32_t lapic_id;      // offset 20
    uint32_t online;        // offset 24
    proc_t  *current;       // offset 32
    proc_t  *idle;          // offset 40
} cpu_local_t;
```

Last reviewed: 2026-07-22
