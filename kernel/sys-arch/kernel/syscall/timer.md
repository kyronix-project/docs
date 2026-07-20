# Timer Operations

This document describes the timer-related syscalls in the Kyronix kernel.

## Syscalls

| Syscall | Number | Description |
|---------|--------|-------------|
| `getrlimit` | 98 | Get resource limits |
| `prlimit64` | 302 | Get/set resource limits |
| `nanosleep` | 35 | Sleep for a specified time |
| `clock_nanosleep` | 230 | Sleep relative to a clock |
| `getitimer` | 36 | Get interval timer value |
| `setitimer` | 38 | Set interval timer |
| `clock_gettime` | 228 | Get clock time |
| `gettimeofday` | 96 | Get time of day |
| `times` | 100 | Get process times |
| `alarm` | 37 | Set an alarm signal |
| `clock_getres` | 229 | Get clock resolution |
| `sysinfo` | 99 | Get system information |
| `getrandom` | 318 | Get random bytes |

## Clock Sources

| Clock ID | Description |
|----------|-------------|
| `CLOCK_REALTIME` | System-wide real-time clock |
| `CLOCK_MONOTONIC` | Monotonic clock (ticks since boot) |
| `CLOCK_PROCESS_CPUTIME_ID` | Per-process CPU time |
| `CLOCK_THREAD_CPUTIME_ID` | Per-thread CPU time |

## nanosleep

`nanosleep(req, rem)` suspends the process for the specified time:

1. Calculate the wakeup tick from the requested duration
2. Set `proc.wakeup_tick`
3. Block the process via `sched_yield_blocking()`
4. On wakeup, return remaining time in `rem`

## clock_gettime

`clock_gettime(clk_id, tp)` returns the current time:

- `CLOCK_REALTIME`: Unix timestamp from RTC + ticks
- `CLOCK_MONOTONIC`: Ticks since boot

## alarm

`alarm(seconds)` sets a timer that sends `SIGALRM` to the process after the specified number of seconds.

## ITIMER

`setitimer(which, new_val, old_val)` sets an interval timer:

- `ITIMER_REAL`: Counts real time, sends `SIGALRM`
- `ITIMER_VIRTUAL`: Counts virtual time (not implemented)
- `ITIMER_PROF`: Counts profiling time (not implemented)

## getrandom

`getrandom(buf, len, flags)` fills a buffer with random bytes from the kernel CSPRNG (ChaCha20-based).
