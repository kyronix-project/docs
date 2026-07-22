# System Architecture

This document describes the high-level architecture of the Kyronix operating system. It is the root of the [System Architecture](sys-arch/index.md) section.

## System Layers

The Kyronix system consists of the following layers, from lowest to highest:

1. **Kernel** -- Hardware abstraction, memory management, process scheduling, syscalls
2. **Drivers** -- PCI, ACPI, AHCI, VirtIO, input, framebuffer, TTY, serial
3. **Filesystems** -- VFS, ext2, procfs, devfs
4. **Networking** -- lwIP TCP/IP stack, virtio-net interface

## Design Principles

- The kernel is hybrid: all basic subsystems run in kernel space with direct hardware access but selected drivers can run in userspace.
- Syscalls follow the Linux x86_64 ABI for compatibility with Linux user-space programs.
- Jail-based sandboxing provides process isolation without requiring separate address spaces.
- The physical memory allocator is lock-free (LL-Free) for scalability on SMP systems.

Last reviewed: 2026-07-22
