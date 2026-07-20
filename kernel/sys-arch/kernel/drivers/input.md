# Input

This document describes the input subsystem in the Kyronix kernel.

## Overview

The input subsystem handles keyboard and mouse input from PS/2 devices. It provides a unified event interface similar to Linux evdev.

## Components

- **PS/2 Keyboard** (`kbd.c`) -- AT-compatible keyboard driver
- **PS/2 Mouse** (`ps2mouse.c`) -- PS/2 mouse/trackpad driver
- **Input Core** (`input.c`) -- event queue and device registration

## Input Events

Input events are reported as `input_event` structures:

```c
struct input_event {
    uint16_t type;   // EV_KEY, EV_REL, EV_SYN
    uint16_t code;   // key code or axis
    int32_t value;   // press/release, movement
};
```

## Keyboard

The PS/2 keyboard driver:

1. Initializes the keyboard controller via port 0x60/0x64
2. Handles IRQ 1 (keyboard interrupt)
3. Translates scan codes to key events
4. Supports key press, release, and repeat
5. Provides key events to `/dev/input/event0`

## Mouse

The PS/2 mouse driver:

1. Initializes the mouse via port 0x60/0x64 with auxiliary device commands
2. Handles IRQ 12 (mouse interrupt)
3. Reads 3-byte or 4-byte packets
4. Reports relative X/Y movement and button state

## TTY Integration

The input subsystem integrates with the TTY layer:

- Keyboard events are forwarded to the active virtual terminal
- Special key combinations (Ctrl+Alt+Del, Ctrl+Alt+F1-F6) trigger console switching
- Terminal escape sequences are generated from key events
