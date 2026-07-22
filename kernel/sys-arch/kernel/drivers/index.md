# Drivers

This document describes the device driver subsystem of the Kyronix kernel. It is the child of [Kernel](sys-arch/kernel/index.md) and parent of driver component documents.

## Components

| Component | Source | Description |
|---|---|---|
| PCI | `drivers/pci.c` | PCI bus enumeration |
| ACPI | `drivers/acpi.c` | ACPI table parsing |
| AHCI | `drivers/ahci.c` | SATA/AHCI block device driver |
| VirtIO | `drivers/virtio_net.c` | VirtIO network device driver |
| Input | `drivers/input.c` | Event-based input subsystem |
| Keyboard | `drivers/kbd.c` | PS/2 keyboard driver |
| PS/2 Mouse | `drivers/ps2mouse.c` | PS/2 mouse driver |
| Framebuffer | `drivers/fb.c` | Linear framebuffer driver |
| fbdev | `drivers/fbdev.c` | Framebuffer device (/dev/fb0) |
| TTY | `drivers/tty.c` | Teletype terminal |
| Virtual TTY | `drivers/vt.c` | Virtual terminal switching |
| Serial | `drivers/serial.c` | Serial port (COM1) |
| Block | `drivers/block.c` | Block device abstraction |
| UIO | `drivers/uio.c` | Userspace I/O device |

## Initialization Order

Drivers are initialized in `kmain()` after PCI enumeration and ACPI setup:

1. `block_init()` -- Block device abstraction
2. `ahci_init()` -- SATA/AHCI controllers
3. `virtnet_init()` -- VirtIO network
4. `net_init()` -- Network stack (lwIP)
5. `uio_init()` -- Userspace I/O
6. `fbdev_init()` -- Framebuffer device
7. `input_init()` -- Input event subsystem
8. `vt_init()` -- Virtual terminal
9. `pit_init()` -- Timer
10. `ps2mouse_init()` -- PS/2 mouse

Last reviewed: 2026-07-22
