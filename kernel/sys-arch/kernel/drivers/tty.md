# TTY

This document describes the TTY (terminal) subsystem in the Kyronix kernel.

## Overview

The TTY subsystem provides terminal emulation, virtual console switching, and serial console support. It manages the interface between user-space applications and the display/input devices.

## Components

- **Virtual Terminals (VT)** (`vt.c`) -- multiple virtual console screens
- **TTY Core** (`tty.c`) -- terminal line discipline and input processing
- **Framebuffer Console** -- renders text to the framebuffer

## Virtual Terminals

Kyronix supports multiple virtual terminals (VT1-VT6), switchable via:

- Ctrl+Alt+F1 through Ctrl+Alt+F6
- Ctrl+Alt+Del for reboot

Each VT maintains its own screen buffer and cursor position.

## Terminal Features

- Text-mode rendering using PSF font (`psf_font.S`)
- Cursor blinking (managed by the timer interrupt)
- ANSI escape sequence parsing
- Scrollback buffer

## Serial Console

When `CONFIG_SERIAL_CONSOLE` is enabled, kernel log output is also sent to the COM1 serial port (UART 16550). This is useful for debugging and headless operation.

## TTY Operations

| Operation | Description |
|-----------|-------------|
| `read()` | Read input from the terminal |
| `write()` | Write output to the terminal |
| `ioctl()` | Terminal control (TIOCGWINSZ, TCGETS, TCSETS, etc.) |
| `pollin()` | Check if input is available |

## Signal Integration

The TTY layer generates signals for terminal events:

- `SIGWINCH` -- window size changed
- `SIGTSTP` -- Ctrl+Z pressed
- `SIGINT` -- Ctrl+C pressed (via line discipline)
