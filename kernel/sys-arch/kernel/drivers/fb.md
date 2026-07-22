# Framebuffer

This document describes the framebuffer driver in the Kyronix kernel. It is the child of [Drivers](sys-arch/kernel/drivers/index.md).

## Source

`kernel/drivers/fb.c`, `kernel/drivers/fbdev.c`

## Overview

The framebuffer driver provides access to the linear framebuffer provided by the Limine bootloader. It supports text rendering via a built-in PSF font and exposes a character device at `/dev/fb0`.

## Initialization

1. Read framebuffer info from Limine (`LIMINE_FRAMEBUFFER_REQUEST`)
2. Map framebuffer into kernel virtual space
3. Clear screen with background color
4. Register `/dev/fb0` character device via fbdev

## Framebuffer Properties

| Field | Description |
|---|---|
| `address` | Linear framebuffer physical address |
| `width`, `height` | Resolution in pixels |
| `pitch` | Bytes per scanline |
| `bpp` | Bits per pixel |

## Text Rendering

The kernel includes a built-in PSF (PC Screen Font) for text rendering. Each character is 8x16 pixels. The framebuffer supports ANSI color codes for colored text output.

## Functions

| Function | Description |
|---|---|
| `fb_init(lfb)` | Initialize framebuffer from Limine response |
| `fb_clear(color)` | Clear screen with solid color |
| `fb_putchar(c)` | Render a character at the current cursor position |
| `fb_set_color(fg, bg)` | Set foreground and background colors |
| `fb_cursor_blink_tick(ticks)` | Update cursor blink state |

Last reviewed: 2026-07-22
