# Signals

This document describes the signal handling implementation in the Kyronix kernel. It is the child of [Process Management](sys-arch/kernel/proc/index.md).

## Source

`kernel/proc/signal.c`, `kernel/proc/signal.h`

## Overview

The Kyronix kernel implements POSIX-compatible signal handling with Linux ABI compatibility. Signals are delivered by constructing an `rt_sigframe` on the user stack and redirecting execution to the signal handler.

## Supported Signals

| Signal | Value | Default Action |
|---|---|---|
| `SIGHUP` | 1 | Fatal |
| `SIGINT` | 2 | Fatal |
| `SIGQUIT` | 3 | Fatal |
| `SIGILL` | 4 | Fatal |
| `SIGTRAP` | 5 | Fatal |
| `SIGABRT` | 6 | Fatal |
| `SIGBUS` | 7 | Fatal |
| `SIGFPE` | 8 | Fatal |
| `SIGKILL` | 9 | Fatal (uncatchable) |
| `SIGUSR1` | 10 | Fatal |
| `SIGSEGV` | 11 | Fatal |
| `SIGUSR2` | 12 | Fatal |
| `SIGPIPE` | 13 | Fatal |
| `SIGALRM` | 14 | Fatal |
| `SIGTERM` | 15 | Fatal |
| `SIGCHLD` | 17 | Non-fatal (ignore) |
| `SIGCONT` | 18 | Non-fatal (continue) |
| `SIGSTOP` | 19 | Stop (uncatchable) |
| `SIGTSTP` | 20 | Stop |
| `SIGTTIN` | 21 | Stop |
| `SIGTTOU` | 22 | Stop |
| `SIGWINCH` | 28 | Non-fatal (ignore) |

## Signal Delivery

### `signal_check(f)`

Called on every syscall entry/exit and timer interrupt:

1. Check terminal-driven signals via `tty_check_signals()`
2. Load pending signals masked by complement of `sig_mask`
3. Find lowest-numbered unmasked signal via `__builtin_ctzll`
4. Clear from `pending_sigs`
5. If ptrace-traced, call `proc_ptrace_stop()`
6. Otherwise call `deliver_signal()`

### `deliver_signal(p, sig, frame)`

- **SIGSTOP/SIGTSTP (default)**: Enter job-control stop
- **SIG_IGN**: No action
- **SIG_DFL**: If fatal, call `proc_do_exit(-sig)`
- **Custom handler**: Build signal frame and redirect execution

### `setup_sigframe(p, sig, frame)`

1. Check for alternate signal stack (`SA_ONSTACK`)
2. Allocate `rt_sigframe_t` (440 bytes) below user RSP
3. Populate with register snapshot, signal info, ucontext
4. Update signal mask (add signal bit + `sa_mask`)
5. Redirect: `rcx` = handler, `rdi` = signal number, `rsi` = siginfo, `rdx` = ucontext

## Signal Action Flags

| Flag | Value | Effect |
|---|---|---|
| `SA_NOCLDSTOP` | 0x0001 | Don't send SIGCHLD on stops |
| `SA_NOCLDWAIT` | 0x0002 | Don't create zombies |
| `SA_SIGINFO` | 0x0004 | Extended handler (3-arg) |
| `SA_ONSTACK` | 0x08000000 | Execute on alternate stack |
| `SA_RESETHAND` | 0x80000000 | Reset to SIG_DFL after delivery |
| `SA_NODEFER` | 0x40000000 | Don't block signal during handler |

## Alternate Signal Stack

Each process can have an alternate stack configured via `sigaltstack()`. The frame is placed at the top of the alternate stack when `SA_ONSTACK` is set.

Last reviewed: 2026-07-22
