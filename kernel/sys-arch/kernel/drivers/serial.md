# Serial

This document describes the serial (UART 16550) driver in the Kyronix kernel.

## Overview

The serial driver provides output via the COM1 UART 16550 port. It is used for kernel log output and as a serial console.

## Port Addresses

| Port | Address | Description |
|------|---------|-------------|
| COM1 | 0x3F8 | Primary serial port |

## Initialization

`serial_init(COM1)` configures the UART:

1. Disable interrupts (write 0x00 to interrupt enable register)
2. Enable DLAB (set baud rate divisor)
3. Set baud rate to 115200 (divisor = 3)
4. 8 data bits, 1 stop bit, no parity (LCR = 0x03)
5. Enable FIFO, clear, 14-byte threshold (FCR = 0xC7)
6. IRQs enabled, RTS/DSR set (MCR = 0x0B)

## Output

`serial_putc(c)` writes a character to the UART:

1. Wait until the transmit holding register is empty (LSR bit 5)
2. Write the character to the data register (port + 0)

## Kernel Log

The serial driver is used by the kernel log system (`log.c`):

- `log_info()`, `log_warn()`, `log_error()`, `log_debug()` all output to serial
- Output is serialized via a spinlock
- Log level filtering is controlled by `CONFIG_LOG_LEVEL`

## Serial Console

When `CONFIG_SERIAL_CONSOLE` is enabled, the serial port serves as the system console. All kernel messages and TTY output are mirrored to serial.
