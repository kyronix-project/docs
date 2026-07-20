# Framebuffer

This document describes the framebuffer driver in the Kyronix kernel.

## Overview

The framebuffer driver provides access to the display via the Limine framebuffer. It supports memory-mapped access for user-space graphical applications.

## Initialization

`fb_init()` is called during boot:

1. Read framebuffer information from the Limine `LIMINE_FRAMEBUFFER_REQUEST`
2. Store framebuffer address, width, height, pitch, and bpp
3. Map the framebuffer memory into kernel space
4. Create `/dev/fb0` device node

## Framebuffer Info

| Field | Description |
|-------|-------------|
| `address` | Physical address of the framebuffer |
| `width` | Width in pixels |
| `height` | Height in pixels |
| `pitch` | Bytes per scanline |
| `bpp` | Bits per pixel (32 typical) |

## Device Interface

The framebuffer device (`/dev/fb0`) supports:

- `mmap` -- map framebuffer memory into user space
- `ioctl` -- query framebuffer parameters

## Framebuffer Device

`fbdev_init()` creates the framebuffer device node with:

- `chr_read` -- read pixel data
- `chr_write` -- write pixel data
- `chr_mmap` -- map framebuffer into user address space

## UIO Support

The UIO (Userspace I/O) driver (`uio_init()`) provides PCI BAR mapping for user-space device drivers. This allows user-space programs to directly access device registers.
