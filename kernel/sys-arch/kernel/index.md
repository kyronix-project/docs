# Kernel

This document describes the Kyronix kernel subsystem. It is the child of [System Architecture](sys-arch/index.md) and parent of all kernel-specific architecture documents.

## Kernel Overview

The Kyronix kernel (codename `k9`) is a monolithic kernel for x86_64, version `0.2`. It boots via the Limine v3 protocol and initializes the following subsystems in order:

1. Serial output and printf setup
2. GDT and IDT
3. Physical Memory Manager (PMM) with LL-Free lock-free allocator
4. Virtual Memory Manager (VMM) with 4-level page tables
5. Local APIC and SMP
6. Kernel heap allocator
7. Syscall entry (SYSCALL/SYSRET)
8. Process scheduler
9. Jail sandboxing
10. Virtual Filesystem (VFS) with `/proc`, `/sys`, `/dev/pts` mounts
11. PCI enumeration, ACPI, AHCI, VirtIO-net
12. Network stack (lwIP)
13. PIT timer and LAPIC timer calibration
14. Application Processor (AP) boot
15. ChaCha20 CSPRNG
16. Root filesystem mount and init exec

## Kernel Entry Point

The kernel entry point is `kmain()` in `kernel/kernel.c:255`. It receives no arguments from the bootloader; all boot information is obtained via Limine protocol requests.

## Sections

- [Boot](boot/index.md) -- Limine protocol and boot sequence
- [Architecture](arch/index.md) -- x86_64 hardware abstractions
- [Memory Management](mm/index.md) -- PMM, VMM, heap, VMA
- [Process Management](proc/index.md) -- Scheduler, SMP, signals, jails
- [Filesystems](fs/index.md) -- VFS, ext2, procfs, devfs
- [Drivers](drivers/index.md) -- PCI, ACPI, AHCI, VirtIO, input, TTY
- [Networking](net/index.md) -- lwIP, network interface
- [Syscalls](syscall/index.md) -- Linux ABI-compatible syscall interface

Last reviewed: 2026-07-22
