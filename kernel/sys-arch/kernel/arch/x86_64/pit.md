# PIT

This document describes the Programmable Interval Timer (PIT) driver in the Kyronix kernel.

## Overview

The PIT is an x86 timer device based on the 8253/8254 chip. It provides a fixed-frequency periodic interrupt used for timekeeping and as a reference for LAPIC timer calibration.

## Configuration

- **Channel:** 0
- **Mode:** 3 (square wave generator)
- **Frequency:** ~250 Hz (divisor 4772, base oscillator 1,193,182 Hz)
- **IRQ:** 0 (mapped to PIC IRQ 0, vector 32)

## Initialization

`pit_init()` performs:

1. Program PIT Channel 0 with mode 0x36 (channel 0, lobyte/hibyte, square wave)
2. Read the CMOS RTC to obtain the current epoch time
3. Unmask IRQ 0 on the PIC

## Tick Counter

A global volatile counter `g_ticks` is incremented on each IRQ 0. This counter serves as the kernel's monotonic time reference.

## RTC Reader

`rtc_read_unix()` reads the CMOS Real-Time Clock registers:

- Handles BCD-to-binary conversion
- Supports 12/24 hour format
- Handles leap year calculations
- Returns a Unix timestamp

The epoch time is used to initialize the system clock at boot.

## Usage

The PIT is used for:

- System timekeeping (tick counting)
- LAPIC timer calibration reference
- Preemption timer (alongside LAPIC timer)
