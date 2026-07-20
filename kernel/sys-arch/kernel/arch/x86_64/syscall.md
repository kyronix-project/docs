# Syscall Entry

This document describes the SYSCALL/SYSRET mechanism used for system calls in the Kyronix kernel.

## Overview

Kyronix uses the x86-64 `SYSCALL`/`SYSRET` instructions for fast system call entry and exit. This mechanism is configured via MSRs and provides a low-overhead transition between user mode (ring 3) and kernel mode (ring 0).

## MSR Configuration

`syscall_init()` configures the following MSRs:

| MSR | Value | Description |
|-----|-------|-------------|
| `IA32_EFER` | SCE=1 | Enable SYSCALL/SYSRET |
| `IA32_STAR` | Kernel CS=0x08, User CS/SS=(GDT_USER_DATA_SEL-0x8)<<48 | Segment selectors for SYSRET |
| `IA32_LSTAR` | Address of `syscall_entry` | RIP target for SYSCALL |
| `IA32_FMASK` | IF, TF, AC, EFLAGS[8:10] cleared | Flags to mask on SYSCALL entry |

## Syscall Stack

A dedicated 16 KiB kernel stack is used for system calls. This stack is set as `rsp0` in the TSS for the BSP CPU.

## Entry Point

The `syscall_entry` assembly routine (`syscall_entry.S`):

```
syscall_entry:
    swapgs                          # switch to kernel GS base
    movq %rsp, %gs:CPU_USER_RSP     # save user RSP
    movq %gs:CPU_KERNEL_RSP, %rsp   # switch to kernel stack
    pushq r15..rax (15 regs)        # save full register state
    movq %rsp, %rdi                 # pass cpu_state_t* as arg
    call syscall_dispatch            # C dispatcher
    popq rax..r15 (15 regs)         # restore registers
    movq %gs:CPU_USER_RSP, %rsp     # restore user RSP
    swapgs                          # switch back to user GS base
    sysretq                         # return to userspace
```

## Syscall Convention

Arguments are passed in the same registers as Linux:

| Register | Argument |
|----------|----------|
| `rax` | Syscall number |
| `rdi` | arg1 |
| `rsi` | arg2 |
| `rdx` | arg3 |
| `r10` | arg4 |
| `r8` | arg5 |
| `r9` | arg6 |

Return value is placed in `rax`.

## Userspace Entry

`enter_userspace` / `enter_userspace_exec` clear all GP registers except `rcx` (RIP), `rdx` (R11=RFLAGS), `rsi` (RSP), then execute `sysretq`. The `_exec` variant additionally performs `swapgs`.

## See Also

- [Syscall Reference](../../syscall/index.md)
