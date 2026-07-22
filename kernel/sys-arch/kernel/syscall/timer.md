# Timers

This document describes the timer syscalls in the Kyronix kernel. It is the child of [Syscalls](index.md).

## Syscall Table

- 35 `nanosleep` - Pause execution for a relative time interval
- 36 `getitimer` - Get current value of an interval timer
- 37 `alarm` - Set a real-time signal alarm
- 38 `setitimer` - Set an interval timer
- 96 `gettimeofday` - Get current wall-clock time
- 97 `getrlimit` - Get resource limits
- 98 `getrusage` - Get resource usage (stub, returns zeroed struct)
- 99 `sysinfo` - Get system information
- 100 `times` - Get process times
- 201 `time` - Get time in seconds
- 228 `clock_gettime` - Get clock time
- 229 `clock_getres` - Get clock resolution
- 230 `clock_nanosleep` - Sleep until an absolute or relative clock time
- 302 `prlimit64` - Get or set resource limits

## Time Model

The Kyronix kernel derives time from two global counters maintained by the Programmable Interval Timer (PIT) and the Real-Time Clock (RTC):

- `g_ticks` (volatile uint64) - Milliseconds since boot, incremented on each PIT tick at approximately 250 Hz
- `g_epoch_base` (uint64) - Unix timestamp in seconds at boot time, read from the RTC during initialization

Wall time in milliseconds is computed as:

```c
wall_ms = g_epoch_base * 1000 + g_ticks
```

The `time` (syscall 201) and `gettimeofday` (syscall 96) syscalls both return wall time derived from this formula. The `clock_gettime` (syscall 228) syscall also uses this computation and returns the result as a `timespec` structure with seconds and nanoseconds.

## Clock Resolution

The `clock_getres` (syscall 229) syscall reports a resolution of {0, 1000000} nanoseconds (1 millisecond). This reflects the PIT tick rate of approximately 250 Hz, yielding a 4-millisecond tick period. The kernel floors sub-millisecond precision to 1 millisecond in all time calculations.

## nanosleep

The `nanosleep` (syscall 35) syscall pauses the calling process for a specified duration:

1. Read the requested time from the user-provided `timespec` structure (seconds and nanoseconds)
2. Convert the duration to milliseconds: `ms = sec * 1000 + nsec / 1000000`
3. Set `p->wakeup_tick = g_ticks + ms`
4. Call `proc_set_timer(p)` to register the timer
5. Yield the processor in a blocking loop until `g_ticks >= deadline` or the process is woken by a signal
6. Clear `p->wakeup_tick` and return 0

## clock_nanosleep

The `clock_nanosleep` (syscall 230) syscall extends `nanosleep` with clock ID and flags support:

1. Read the requested time from the user-provided `timespec` structure
2. If the `TIMER_ABSTIME` flag (bit 0) is set, compute the relative sleep duration as `target_ms - current_wall_ms`
3. Otherwise, treat the request as a relative sleep duration
4. Follow the same blocking loop as `nanosleep`

## alarm

The `alarm` (syscall 37) syscall sets a real-time signal alarm:

1. Compute the previous alarm's remaining seconds from `p->alarm_tick`
2. If `seconds > 0`, set `p->alarm_tick = g_ticks + seconds * 1000` and register the timer
3. If `seconds == 0`, clear `p->alarm_tick`
4. Return the number of seconds remaining on the previous alarm

The alarm delivers `SIGALRM` to the process when the deadline expires. The kernel checks `p->alarm_tick` in the PIT interrupt handler (IDT vector 32).

## itimer (Interval Timer)

The `setitimer` (syscall 38) and `getitimer` (syscall 36) syscalls manage interval timers that deliver `SIGALRM` at periodic intervals:

1. `setitimer` reads the new interval from a `itimerval` structure containing interval (seconds, microseconds) and value (seconds, microseconds)
2. The interval is stored in `p->itimer_interval_ms` and the next trigger in `p->itimer_next_tick`
3. `getitimer` returns the current interval and remaining time until the next trigger
4. The kernel checks `p->itimer_next_tick` in the PIT interrupt handler and delivers `SIGALRM` when the deadline expires

## getrlimit / prlimit64

The `getrlimit` (syscall 97) and `prlimit64` (syscall 302) syscalls report resource limits:

| Resource | Hard Limit | Soft Limit |
|---|---|---|
| `RLIMIT_NOFILE` (7) | `VFS_FD_MAX` (1024) | `VFS_FD_MAX` (1024) |
| All others | 1 GiB (1073741824) | 1 GiB (1073741824) |

The `prlimit64` syscall accepts a PID argument but ignores it, always operating on the current process. The new limits argument (`nl`) is accepted but not applied.

## sysinfo

The `sysinfo` (syscall 99) syscall returns system information in a fixed structure:

| Field | Value |
|---|---|
| uptime | `g_ticks / 1000` (seconds since boot) |
| totalram | 256 MiB (268435456 bytes) |
| freeram | 128 MiB (134217728 bytes) |
| mem_unit | 1 (byte) |
| procs | 1 |

All other fields are zeroed.

## times

The `times` (syscall 100) syscall returns a monotonic tick count (`g_ticks + 1`). The user-provided `tms` structure is zeroed. The return value is always positive.

Last reviewed: 2026-07-22
