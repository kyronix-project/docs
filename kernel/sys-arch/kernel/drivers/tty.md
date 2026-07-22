# TTY

This document describes the TTY (teletype) subsystem in the Kyronix kernel. It is the child of [Drivers](sys-arch/kernel/drivers/index.md).

## Source

`kernel/drivers/tty.c`, `kernel/drivers/vt.c`

## Overview

The TTY subsystem provides terminal I/O with support for multiple virtual terminals. It handles character buffering, line editing, and ANSI escape sequence processing.

## Components

| Component | Source | Description |
|---|---|---|
| TTY | `tty.c` | Terminal I/O and line discipline |
| Virtual TTY | `vt.c` | Virtual terminal switching |

## Features

- Line buffering with echo
- ANSI escape sequence parsing (colors, cursor movement)
- Multiple virtual terminals (switched via keyboard)
- SIGINT (Ctrl+C), SIGQUIT (Ctrl+\) generation
- Cursor blink timer

## Functions

| Function | Description |
|---|---|
| `tty_putchar(c)` | Output a character to the current terminal |
| `tty_check_signals()` | Check for terminal-driven signals |

## Signal Generation

| Key | Signal | Description |
|---|---|---|
| Ctrl+C | SIGINT | Interrupt |
| Ctrl+\ | SIGQUIT | Quit |
| Ctrl+Z | SIGTSTP | Terminal stop |

Last reviewed: 2026-07-22
