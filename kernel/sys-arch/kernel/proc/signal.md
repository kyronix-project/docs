# Signal Handling

This document describes the POSIX signal handling subsystem in the Kyronix kernel.

## Overview

Kyronix implements POSIX.1 real-time signals with support for signal handlers, signal masks, alternate signal stacks, and job control stops.

## Signal Numbers

Standard POSIX signals are supported (1-31), including:

| Signal | Default Action | Description |
|--------|---------------|-------------|
| SIGHUP (1) | Terminate | Hangup |
| SIGINT (2) | Terminate | Interrupt |
| SIGQUIT (3) | Core dump | Quit |
| SIGILL (4) | Core dump | Illegal instruction |
| SIGTRAP (5) | Core dump | Trap |
| SIGABRT (6) | Core dump | Abort |
| SIGBUS (7) | Core dump | Bus error |
| SIGFPE (8) | Core dump | Floating point exception |
| SIGKILL (9) | Terminate | Kill (unblockable) |
| SIGUSR1 (10) | Terminate | User-defined signal 1 |
| SIGSEGV (11) | Core dump | Segmentation fault |
| SIGUSR2 (12) | Terminate | User-defined signal 2 |
| SIGPIPE (13) | Terminate | Broken pipe |
| SIGALRM (14) | Terminate | Alarm |
| SIGTERM (15) | Terminate | Termination |
| SIGCHLD (18) | Ignore | Child exited |
| SIGCONT (19) | Continue | Continue |
| SIGSTOP (20) | Stop | Stop (unblockable) |
| SIGTSTP (21) | Stop | Terminal stop |
| SIGWINCH (28) | Ignore | Window resize |

## Signal Delivery

`proc_send_signal(proc, sig)` sets the corresponding bit in `pending_sigs` and wakes the process if it is in `PROC_WAITING` state.

`signal_check(frame)` is called from the syscall return path:

1. Checks TTY signals (SIGWINCH, SIGTSTP, etc.)
2. Finds the lowest unmasked pending signal
3. Clears the pending bit
4. If the process is being traced (ptrace), stops for the tracer
5. Otherwise, delivers the signal

## Signal Actions

`rt_sigaction(sig, act, oldact)` sets/gets signal disposition:

| Handler | Behavior |
|---------|----------|
| `SIG_DFL` | Default action (terminate, core, ignore, stop, continue) |
| `SIG_IGN` | Ignore the signal |
| User handler | Set up signal frame and jump to handler |

## Signal Frame

When a user handler is invoked, the kernel sets up an `rt_sigframe_t` on the user stack containing:

- `uc.uc_mcontext` -- full register context (256 bytes)
- `uc.uc_sigmask` -- previous signal mask
- `info.si_signo` -- signal number
- `pretcode` -- address of the signal restorer

The handler is called with:

- `rdi` = signal number
- `rsi` = pointer to `siginfo_t`
- `rdx` = pointer to `ucontext_t`

## Signal Return

`rt_sigreturn()` restores the full register context from the signal frame on the user stack, including the previous signal mask.

## Signal Mask

- `rt_sigprocmask(how, set, oldset)` with `SIG_BLOCK`, `SIG_UNBLOCK`, `SIG_SETMASK`
- `SIGKILL` and `SIGSTOP` cannot be masked
- During signal delivery, the signal's own mask (plus `sa_mask`) is applied

## Alternate Signal Stack

`sigaltstack()` allows a process to specify an alternate stack for signal handlers. If `SA_ONSTACK` is set and the alternate stack is configured, the signal frame is placed on the alternate stack.

## Job Control

- `SIGSTOP` / `SIGTSTP` stop the process (enters `PROC_STOPPED`)
- `SIGCONT` resumes a stopped process
- Stopped processes notify their parent via `SIGCHLD`
