# Summary

[Introduction](getting-started/index.md)

---

# Building

- [Building](building/index.md)
  - [with cbuildrt](building/with-cbuildrt.md)
  - [with Docker/Podman](building/with-docker.md)
  - [without containers](building/without-containers.md)

# Contributing

- [Contributing](contributing/index.md)
  - [Overview](contributing/overview.md)
  - [Commit Messages](contributing/commit-messages.md)
  - [Coding Style](contributing/coding-style.md)
  - [AI Policy](contributing/ai-policy.md)

---

# System Architecture

- [Architecture](sys-arch/index.md)
- [Components](sys-arch/components.md)

## Kernel

- [Kernel](sys-arch/kernel/index.md)

### Boot

- [Boot](sys-arch/kernel/boot/index.md)
  - [Limine Protocol](sys-arch/kernel/boot/limine.md)

### Architecture (x86-64)

- [Architecture](sys-arch/kernel/arch/index.md)
  - [x86-64](sys-arch/kernel/arch/x86_64/index.md)
  - [CPU Primitives](sys-arch/kernel/arch/x86_64/cpu.md)
  - [GDT](sys-arch/kernel/arch/x86_64/gdt.md)
  - [IDT](sys-arch/kernel/arch/x86_64/idt.md)
  - [LAPIC](sys-arch/kernel/arch/x86_64/lapic.md)
  - [PIT](sys-arch/kernel/arch/x86_64/pit.md)
  - [Syscall Entry](sys-arch/kernel/arch/x86_64/syscall.md)

### Memory Management

- [Memory Management](sys-arch/kernel/mm/index.md)
  - [PMM](sys-arch/kernel/mm/pmm.md)
  - [VMM](sys-arch/kernel/mm/vmm.md)
  - [Heap](sys-arch/kernel/mm/heap.md)
  - [VMA](sys-arch/kernel/mm/vma.md)

### Process Management

- [Process Management](sys-arch/kernel/proc/index.md)
  - [Scheduler](sys-arch/kernel/proc/scheduler.md)
  - [SMP](sys-arch/kernel/proc/smp.md)
  - [Signals](sys-arch/kernel/proc/signal.md)
  - [Jail](sys-arch/kernel/proc/jail.md)

### Filesystems

- [Filesystems](sys-arch/kernel/fs/index.md)
  - [VFS](sys-arch/kernel/fs/vfs.md)
  - [ext2](sys-arch/kernel/fs/ext2.md)
  - [procfs](sys-arch/kernel/fs/procfs.md)
  - [devfs](sys-arch/kernel/fs/devfs.md)
  - [eventfd](sys-arch/kernel/fs/eventfd.md)
  - [Pipe](sys-arch/kernel/fs/pipe.md)

### Drivers

- [Drivers](sys-arch/kernel/drivers/index.md)
  - [PCI](sys-arch/kernel/drivers/pci.md)
  - [ACPI](sys-arch/kernel/drivers/acpi.md)
  - [AHCI](sys-arch/kernel/drivers/ahci.md)
  - [VirtIO](sys-arch/kernel/drivers/virtio.md)
  - [Input](sys-arch/kernel/drivers/input.md)
  - [Framebuffer](sys-arch/kernel/drivers/fb.md)
  - [TTY](sys-arch/kernel/drivers/tty.md)
  - [Serial](sys-arch/kernel/drivers/serial.md)

### Networking

- [Networking](sys-arch/kernel/net/index.md)
  - [lwIP](sys-arch/kernel/net/lwip.md)
  - [Network Interface](sys-arch/kernel/net/netif.md)

### Syscalls

- [Syscalls](sys-arch/kernel/syscall/index.md)
  - [File Operations](sys-arch/kernel/syscall/file.md)
  - [Process Control](sys-arch/kernel/syscall/process.md)
  - [Memory Management](sys-arch/kernel/syscall/memory.md)
  - [Socket Operations](sys-arch/kernel/syscall/socket.md)
  - [Timers](sys-arch/kernel/syscall/timer.md)
  - [Epoll](sys-arch/kernel/syscall/epoll.md)
  - [Futex](sys-arch/kernel/syscall/futex.md)
  - [Ptrace](sys-arch/kernel/syscall/ptrace.md)

## Libraries

- [Libraries](sys-arch/lib/index.md)
  - [Frigg](sys-arch/lib/frigg.md)
  - [Bragi](sys-arch/lib/bragi.md)
  - [Hel](sys-arch/lib/hel.md)
  - [Helix](sys-arch/lib/helix.md)
  - [libasync](sys-arch/lib/libasync.md)
  - [C Library](sys-arch/lib/libc/index.md)
    - [mlibc](sys-arch/lib/libc/mlibc.md)
    - [Syscall Wrappers](sys-arch/lib/libc/syscalls.md)

## Servers

- [Servers](sys-arch/servers/index.md)
  - [Init](sys-arch/servers/init.md)
  - [mbus](sys-arch/servers/mbus.md)
  - [POSIX Subsystem](sys-arch/servers/posix-subsystem.md)
  - [udev](sys-arch/servers/udev.md)

## User-Space Drivers

- [Drivers](sys-arch/drivers/index.md)
  - [libblockfs](sys-arch/drivers/libblockfs.md)

---

# Implementation Notes

- [Implementation Notes](impl-notes/index.md)

## Kernel

- [Kernel](impl-notes/kernel/index.md)
  - [Initialization](impl-notes/kernel/initialization.md)
  - [Scheduling](impl-notes/kernel/scheduling.md)
  - [IRQ Handling](impl-notes/kernel/irq-handling.md)
  - [Memory Management](impl-notes/kernel/memory-management.md)

## Drivers

- [Drivers](impl-notes/drivers/index.md)
  - [PCI Enumeration](impl-notes/drivers/pci.md)

---

# Reference

- [Reference](reference/index.md)
  - [Syscalls](reference/syscalls.md)
  - [Protocols](reference/protocols.md)
  - [API](reference/api.md)
