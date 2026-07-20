# devfs

This document describes the devfs virtual filesystem in the Kyronix kernel.

## Overview

devfs provides a virtual filesystem at `/dev` that exposes device nodes. It is similar to Linux's devtmpfs but simpler.

## Device Nodes

| Path | Type | Description |
|------|------|-------------|
| `/dev/null` | char | Discards all writes, returns EOF on read |
| `/dev/zero` | char | Returns zero bytes on read |
| `/dev/random` | char | Returns random bytes (CSPRNG) |
| `/dev/urandom` | char | Returns random bytes (CSPRNG) |
| `/dev/tty` | char | Controlling terminal |
| `/dev/console` | char | System console |
| `/dev/fb0` | char | Framebuffer device |
| `/dev/input/event0` | char | Input event device |
| `/dev/sda` | char | Block device (SATA/AHCI) |

## Implementation

devfs creates device nodes during `vfs_init()` using `vfs_create_chr()`. Each device node has associated character device operations:

- `chr_read` -- read from device
- `chr_write` -- write to device
- `chr_ioctl` -- device-specific control
- `chr_pollin` -- check for available input
- `chr_mmap` -- memory-map device (for framebuffer, UIO)

## Framebuffer Device

The framebuffer device (`/dev/fb0`) supports:

- `mmap` -- map framebuffer memory into user space
- `ioctl` -- query framebuffer parameters (resolution, bpp, stride)

## Input Device

The input device (`/dev/input/event0`) provides evdev-style input events:

- `read` -- returns `input_event` structures (type, code, value)
- `pollin` -- reports whether events are available
