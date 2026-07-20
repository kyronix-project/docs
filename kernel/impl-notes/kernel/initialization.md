# Initialization

This document describes the boot sequence and subsystem initialization in the Kyronix kernel.

## Boot Sequence

The kernel boots via the Limine protocol v3. Limine calls `kmain()` after loading the kernel ELF and setting up the environment.

### Phase 1: Early Hardware Setup

1. **Serial console** (`serial_init(COM1)`) -- Initialize UART 16550 at 115200 baud
2. **GDT** (`gdt_init()`) -- Set up Global Descriptor Table with kernel/user segments and TSS
3. **IDT** (`idt_init()`) -- Set up Interrupt Descriptor Table, remap PIC to vectors 32-47
4. **Keyboard** (`kbd_init()`) -- Initialize PS/2 keyboard controller

### Phase 2: Memory Initialization

5. **PMM** (`pmm_init(mmap, hhdm)`) -- Initialize bitmap-based physical page allocator from Limine memory map
6. **Framebuffer** (`fb_init()`) -- Store framebuffer info from Limine
7. **VMM** (`vmm_init()`) -- Enable NX bit, initialize kernel address space

### Phase 3: Multiprocessor and Advanced Features

8. **LAPIC** (`lapic_init()`) -- Map Local APIC MMIO, enable it
9. **SMP** (`smp_init()`) -- Discover CPUs from Limine SMP response
10. **SMEP** -- Enable Supervisor Mode Execution Prevention (CR4 bit 20, if supported)
11. **Heap** (`heap_init()`) -- Initialize kernel heap at 0xffff910000000000

### Phase 4: System Services

12. **Syscalls** (`syscall_init()`) -- Configure SYSCALL/SYSRET MSRs, enable SSE
13. **Process table** (`proc_init()`) -- Initialize 64-slot process table
14. **Jails** (`jail_init()`) -- Initialize jail subsystem
15. **VFS** (`vfs_init()`) -- Create root filesystem tree, mount /proc, /dev

### Phase 5: Device Drivers

16. **PCI** (`pci_enumerate()`) -- Scan PCI bus, discover devices
17. **ACPI** (`acpi_init(rsdp)`) -- Parse ACPI tables
18. **Block devices** (`block_init()`, `ahci_init()`) -- Initialize SATA/AHCI
19. **Network** (`virtnet_init()`, `net_init()`) -- VirtIO-net + lwIP

### Phase 6: Userspace I/O

20. **UIO** (`uio_init()`) -- Userspace I/O PCI BAR mapping
21. **Framebuffer device** (`fbdev_init()`) -- Create /dev/fb0
22. **Input** (`input_init()`) -- Initialize input event system
23. **TTY** (`vt_init()`) -- Initialize virtual terminals

### Phase 7: Timers and SMP Boot

24. **PIT** (`pit_init()`) -- Program PIT at ~250 Hz, read RTC for epoch
25. **LAPIC timer** (`lapic_calibrate_timer()`) -- Calibrate against PIT
26. **AP boot** (`smp_boot_aps()`) -- Boot Application Processors

### Phase 8: Random and Self-Tests

27. **CSPRNG** -- Initialize ChaCha20-based CSPRNG from RDRAND or RDTSC
28. **Self-tests** -- Run PMM/VMM/heap self-tests

### Phase 9: Root Filesystem and Init

29. **Root FS** -- Mount ext2 from disk or CPIO initrd
30. **Execute /init** -- Run `/init` (or `/sbin/init` or `/bin/init`)
31. **Halt** -- BSP enters idle loop

## Configuration

The kernel is configured via `kernel/config.h` (generated from Kconfig):

| Option | Default | Description |
|--------|---------|-------------|
| `CONFIG_KMEMLEAK` | y | Kernel memory leak detector |
| `CONFIG_SERIAL_CONSOLE` | y | Serial console output |
| `CONFIG_LOG_LEVEL` | 1 | Log level (0=error, 1=warn, 2=info, 3=debug) |
