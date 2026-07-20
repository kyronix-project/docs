# System Components

This document provides a high-level overview of all major components in the Kyronix operating system.

## Overview

Kyronix is a monolithic kernel operating system with a Linux-compatible syscall interface. The system is organized into the following layers:

```
+------------------------------------------+
|             User Applications            |
+------------------------------------------+
|     POSIX Subsystem  |  Other Servers    |
+------------------------------------------+
|          Kyronix Kernel (monolithic)     |
|  +------+------+------+------+------+   |
|  | mm   | proc | fs   | driver| net  |   |
|  +------+------+------+------+------+   |
+------------------------------------------+
|         Hardware (x86-64)                |
+------------------------------------------+
```

## Kernel Subsystems

| Subsystem | Description |
|-----------|-------------|
| **mm** | Physical and virtual memory management, heap allocator, VMAs |
| **proc** | Process management, scheduling, SMP, signals, jail sandboxing |
| **fs** | VFS, ext2, FAT32, procfs, devfs, pipes, eventfd, sockets |
| **drivers** | PCI, ACPI, AHCI, VirtIO, input, framebuffer, TTY, serial |
| **net** | lwIP-based networking stack with VirtIO-net backend |
| **syscall** | 150+ Linux-compatible syscall implementations |

## User-Space Components

| Component | PID | Description |
|-----------|-----|-------------|
| **init** | 1 | Init process, mounts root filesystem, starts services |
| **ksh** | -- | POSIX shell |
| **posix-subsystem** | -- | POSIX compatibility layer |
| **mbus** | -- | Message bus for device discovery |
| **udev** | -- | Device manager |

## Build System

The kernel is built using GNU Make with two modes:

- **Native build:** Direct compilation on the host (requires musl-gcc, nasm, xorriso)
- **Container build:** Reproducible builds via Podman/Docker (Alpine 3.20 base image)

The build produces:

- `dist/kernel.elf` -- linked kernel binary
- `build/bin/` -- statically-linked user-space binaries
- `initrd.cpio` -- CPIO root filesystem archive
- `dist/kyronix.iso` -- bootable ISO image (BIOS+UEFI)

## Boot Process

1. Limine bootloader loads `kernel.elf` and optional initrd module
2. Limine calls `kmain()` at virtual address `0xffffffff80000000`
3. Kernel initializes serial, GDT, IDT, memory, LAPIC, SMP
4. Root filesystem is mounted (ext2 or CPIO initrd)
5. `/init` (or `/sbin/init`, `/bin/init`) is executed

See [Boot](kernel/boot/index.md) for details.
