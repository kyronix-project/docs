# Components

This document lists the major components of the Kyronix kernel and their source locations.

## Kernel Subsystems

| Component | Source Path | Description |
|---|---|---|
| Architecture (x86_64) | `kernel/arch/x86_64/` | CPU primitives, GDT, IDT, LAPIC, PIT, syscall entry |
| Memory Management | `kernel/mm/` | PMM (LL-Free), VMM, VMA, heap, shared memory |
| Process Management | `kernel/proc/` | Process table, scheduler, SMP, signals, jails |
| Filesystems | `kernel/fs/` | VFS, ext2, FAT32, procfs, devfs, pipes, sockets |
| Drivers | `kernel/drivers/` | PCI, ACPI, AHCI, VirtIO, input, framebuffer, TTY, serial |
| Syscalls | `kernel/syscall/` | Linux ABI-compatible syscall dispatcher and handlers |
| Networking | `kernel/net/` | lwIP integration, virtio-net interface |
| Cryptography | `kernel/crypto/` | ChaCha20 CSPRNG |
| Executable Loading | `kernel/exec/` | ELF loader, process exec, stack setup |
| Boot | `kernel/boot/` | Limine protocol definitions |

## Libraries

| Library | Source Path | Description |
|---|---|---|
| String | `kernel/lib/string.c` | libc-style string functions |
| Printf | `kernel/lib/printf.c` | Kernel printf implementation |
| Log | `kernel/lib/log.c` | Kernel logging (log_info, log_warn) |
| Kallsyms | `kernel/lib/kallsyms.c` | Kernel symbol lookup |

Last reviewed: 2026-07-22
