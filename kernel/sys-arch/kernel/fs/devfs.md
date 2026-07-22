# devfs

This document describes the devfs filesystem in the Kyronix kernel. It is the child of [Filesystems](sys-arch/kernel/fs/index.md).

## Source

`kernel/fs/devfs.c`

## Overview

devfs is an in-memory filesystem mounted at `/dev` that provides device nodes. It dynamically creates entries for registered character and block devices.

## Mount Points

| Mount | Description |
|---|---|
| `/dev` | Device nodes |
| `/sys` | System information |
| `/dev/pts` | Pseudo-terminal devices |

## Device Registration

Drivers register device nodes via `devfs_register()`:

| Device | Path | Description |
|---|---|---|
| Framebuffer | `/dev/fb0` | Linear framebuffer device |
| Input event 0 | `/dev/input/event0` | Keyboard input |
| Input event 1 | `/dev/input/event1` | Mouse input |
| AHCI disk | `/dev/ahci0` | SATA/AHCI block device |

## Device Types

| Type | Description |
|---|---|
| `VFS_TYPE_DEV` | Character device |
| `VFS_TYPE_BLK` | Block device |

Last reviewed: 2026-07-22
