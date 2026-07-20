# Kernel

This section describes the internal architecture of the Kyronix kernel. The kernel is organized into the following subsystems:

## Subsystems

- [Boot](boot/index.md) -- bootloader protocol and early initialization
- [Architecture](arch/index.md) -- x86-64 specific code (CPU, GDT, IDT, LAPIC, PIT, syscall entry)
- [Memory Management](mm/index.md) -- physical/virtual memory, heap, VMAs
- [Process Management](proc/index.md) -- scheduler, SMP, signals, jail sandboxing
- [Filesystems](fs/index.md) -- VFS, ext2, procfs, devfs, pipes, eventfd
- [Drivers](drivers/index.md) -- PCI, ACPI, AHCI, VirtIO, input, framebuffer, TTY, serial
- [Networking](net/index.md) -- lwIP integration and VirtIO-net
- [Syscalls](syscall/index.md) -- Linux-compatible syscall implementations

## Initialization Order

The kernel initializes subsystems in the following order during boot:

1. Serial console (`serial_init`)
2. GDT and TSS (`gdt_init`)
3. IDT and PIC remapping (`idt_init`)
4. PS/2 keyboard (`kbd_init`)
5. Physical memory manager (`pmm_init`)
6. Framebuffer (`fb_init`)
7. Virtual memory manager (`vmm_init`)
8. Local APIC (`lapic_init`)
9. SMP discovery (`smp_init`)
10. SMEP enable (if supported)
11. Kernel heap (`heap_init`)
12. Syscall MSRs (`syscall_init`)
13. Process table (`proc_init`)
14. Jail subsystem (`jail_init`)
15. Virtual filesystem (`vfs_init`)
16. PCI enumeration (`pci_enumerate`)
17. ACPI (`acpi_init`)
18. Block devices (`block_init`, `ahci_init`)
19. Network (`virtnet_init`, `net_init`)
20. Userspace I/O, framebuffer device, input, TTY
21. PIT timer (`pit_init`)
22. LAPIC timer calibration (`lapic_calibrate_timer`)
23. AP boot (`smp_boot_aps`)
24. CSPRNG initialization
25. Root filesystem mount and `/init` execution
