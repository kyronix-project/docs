# Getting Started

This document provides an overview of the Kyronix's kernel and instructions for building and running it.

## What is Kyronix's kernel

Kyronix's kernel(aka `k9`) is a POSIX-like operating system kernel for the x86_64 architecture. It implements a hybrid design with a custom jail-based sandboxing subsystem, lwIP-based networking, and an ext2 root filesystem.

## Key Features

- Limine v3 boot protocol support (BIOS and UEFI)
- Lock-free physical memory allocator (LL-Free)
- Virtual memory with 4-level page tables and demand paging
- Process management with round-robin scheduling
- SMP support (up to `MAX_CPUS` cores)
- POSIX signals, file descriptors, and process management
- Jail-based sandboxing with filesystem, PID, IPC, and privilege isolation
- lwIP TCP/IP stack over virtio-net
- ext2, FAT32, and CPIO filesystems
- ChaCha20-based CSPRNG
- Kernel memory leak detector (`kmemleak`)

## Quick Start

1. Build the kernel and bootable ISO:

```bash
make iso
```

2. Run in QEMU with KVM acceleration:

```bash
make run
```

3. For serial-only output:

```bash
make run-serial
```

## Architecture

The kernel targets x86_64 and uses the following design:

- **Boot**: Limine v3 protocol provides memory map, framebuffer, HHDM, RSDP, and kernel address information.
- **Memory**: LL-Free lock-free allocator for physical frames; 4-level page tables for virtual memory; first-fit heap allocator.
- **Scheduling**: Per-CPU round-robin with lock-free bitmap scanning via `g_ready_mask`.
- **Syscalls**: Linux x86_64 ABI-compatible syscall interface via `SYSCALL`/`SYSRET`.
- **Networking**: lwIP stack connected to virtio-net with static QEMU user-mode IP (10.0.2.15/24).

## Directory Structure

| Path | Description |
|---|---|
| `kernel/` | Kernel source (arch, mm, fs, drivers, syscall, proc, net) |
| `user/` | Userspace programs (shell, utilities) |
| `rootfs/` | Root filesystem template |
| `scripts/` | Build scripts and Kconfig tools |
| `limine/` | Limine bootloader files |
| `dist/` | Build output directory |

## Testing

Run the full test suite:

```bash
make test-iso && make test-run-log
```

This boots the kernel with a `testrunner` init program, executes test binaries, and reports pass/fail status via serial output. Memory leak detection is performed via the `kmemleak` subsystem.

Last reviewed: 2026-07-22
