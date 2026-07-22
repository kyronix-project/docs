# Input

This document describes the input subsystem in the Kyronix kernel. It is the child of [Drivers](sys-arch/kernel/drivers/index.md).

## Source

`kernel/drivers/input.c`, `kernel/drivers/kbd.c`, `kernel/drivers/ps2mouse.c`

## Overview

The input subsystem provides an event-based interface for keyboard and mouse input. Events are delivered to user-space via character devices at `/dev/input/event0` (keyboard) and `/dev/input/event1` (mouse).

## Event Structure

```c
struct input_event {
    uint32_t type;      // Event type
    uint32_t code;      // Key/button code
    int32_t  value;     // Value (press/release/movement)
};
```

## Event Types

| Type | Description |
|---|---|
| Keyboard event | Key press/release with scancode |
| Mouse event | Relative X/Y movement and button state |

## Devices

| Device | Path | Source |
|---|---|---|
| Keyboard | `/dev/input/event0` | PS/2 keyboard |
| Mouse | `/dev/input/event1` | PS/2 mouse |

## Functions

| Function | Description |
|---|---|
| `input_init()` | Initialize input subsystem |
| `kbd_init()` | Initialize PS/2 keyboard |
| `ps2mouse_init()` | Initialize PS/2 mouse |

## Terminal Integration

The input subsystem is connected to the TTY layer. Keyboard events drive the virtual terminal, and mouse events are forwarded to user-space via the event devices.

Last reviewed: 2026-07-22
