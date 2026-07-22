# Serial

This document describes the serial port driver in the Kyronix kernel. It is the child of [Drivers](sys-arch/kernel/drivers/index.md).

## Source

`kernel/drivers/serial.c`

## Overview

The serial driver provides early kernel output via the COM1 serial port (I/O address `0x3F8`). It is the first driver initialized during boot and remains available for debug output throughout kernel execution.

## COM1 Port Layout

| Port | Offset | Description |
|---|---|---|
| `0x3F8` | +0 | Data register (read/write) |
| `0x3F9` | +1 | Interrupt Enable Register |
| `0x3FA` | +2 | FIFO Control Register |
| `0x3FB` | +3 | Line Control Register |
| `0x3FC` | +4 | Modem Control Register |
| `0x3FD` | +5 | Line Status Register |

## Initialization

```c
serial_init(COM1);
```

1. Disable interrupts (write 0 to IER)
2. Enable DLAB (set LCR bit 7)
3. Set baud rate divisor to 1 (115200 baud)
4. 8 data bits, 1 stop bit, no parity (LCR = 0x03)
5. Enable FIFO, clear, 14-byte threshold (FCR = 0xC7)
6. IRQs enabled, RTS/DSR set (MCR = 0x0B)

## Output

Serial output is used for:

- Kernel debug messages (`log_info`, `log_warn`)
- Boot status messages
- Test framework output (`test-run-log`)
- QEMU serial console (`-serial stdio`)

NOTE: The serial port is initialized before any other driver, making it the most reliable output path for early boot debugging.

Last reviewed: 2026-07-22
