# PIT

This document describes the Programmable Interval Timer (PIT) implementation in the Kyronix kernel. It is the child of [x86-64 Architecture](sys-arch/kernel/arch/x86_64/index.md).

## Source

`kernel/arch/x86_64/pit.c`

## Overview

The PIT provides the system tick source via channel 0. The CMOS Real-Time Clock (RTC) is read at boot to establish a Unix epoch base time.

## PIT Configuration

| Setting | Value |
|---|---|
| Channel | 0 |
| Mode | Square wave (mode 3) |
| Reload value | 4772 |
| Frequency | `1193182 / 4772 = ~250.06 Hz` |
| Tick interval | ~4 ms |

## Key Functions

| Function | Description |
|---|---|
| `pit_init()` | Program PIT channel 0, read RTC epoch, unmask PIC IRQ 0 |
| `pit_read_counter()` | Latch and read PIT channel 0 counter (used during LAPIC calibration) |

## RTC Reading

The CMOS RTC is read at boot to obtain the current date/time:

1. Wait for UIP (Update In Progress) flag to clear (CMOS register 0x0A, bit 7)
2. Read seconds, minutes, hours, day, month, year, century from CMOS registers
3. Convert BCD to binary if needed (Status Register B, bit 2)
4. Handle 12/24 hour mode (Status Register B, bit 1)
5. Compute Unix timestamp: total seconds since 1970-01-01

The resulting `g_epoch_base` is used by `sys_gettimeofday()`, `sys_clock_gettime()`, and `sys_time()` to provide wall-clock time.

## Global State

| Variable | Type | Description |
|---|---|---|
| `g_ticks` | `volatile uint64_t` | System tick counter (incremented on each IRQ 0) |
| `g_epoch_base` | `uint64_t` | Unix timestamp at boot |

NOTE: The PIT is the legacy timer source. The LAPIC timer is calibrated against it and provides the per-CPU scheduling tick at 250 Hz.

Last reviewed: 2026-07-22
