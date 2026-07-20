# Drivers

This section describes the device drivers in the Kyronix kernel.

## Components

- [PCI](pci.md) -- PCI bus enumeration and configuration
- [ACPI](acpi.md) -- ACPI power management
- [AHCI](ahci.md) -- SATA/AHCI block device driver
- [VirtIO](virtio.md) -- VirtIO device support (network)
- [Input](input.md) -- PS/2 keyboard and mouse input
- [Framebuffer](fb.md) -- Linear framebuffer display
- [TTY](tty.md) -- Terminal emulation and virtual consoles
- [Serial](serial.md) -- UART 16550 serial console

## Driver Model

Kyronix uses a simple, direct driver model without a complex device tree or driver framework. Drivers are initialized in a fixed order during boot:

1. PCI enumeration
2. ACPI
3. AHCI (block devices)
4. VirtIO-net (network)
5. Framebuffer
6. Input (PS/2 keyboard, mouse)
7. TTY / virtual consoles
8. Serial (UART 16550)

## Interrupt Handling

Hardware interrupts are dispatched through the PIC (vectors 32-47) or LAPIC. Each IRQ has a registered handler in `g_irq_handlers[irq]`. The handler is called from the common ISR path after register state is saved.
