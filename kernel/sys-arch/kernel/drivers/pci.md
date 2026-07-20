# PCI

This document describes the PCI bus enumeration and configuration in the Kyronix kernel.

## Overview

The PCI driver scans the PCI bus to discover and configure hardware devices. It uses the standard x86 PCI configuration mechanism (ports 0xCF8/0xCFC).

## Configuration Space Access

| Function | Description |
|----------|-------------|
| `pci_read32(bus, dev, fn, reg)` | Read a 32-bit PCI config register |
| `pci_write32(bus, dev, fn, reg, val)` | Write a 32-bit PCI config register |
| `pci_read16(bus, dev, fn, reg)` | Read a 16-bit PCI config register |
| `pci_read8(bus, dev, fn, reg)` | Read an 8-bit PCI config register |

## Device Structure

```c
typedef struct {
    uint8_t bus, dev, fn;
    uint16_t vendor, device;
    uint8_t class, subclass, prog_if;
    uint64_t bars[6];      // physical base of each BAR
    uint32_t bar_sizes[6]; // size of each BAR region
    uint8_t irq_line;
    uint8_t irq_pin;
    uint8_t header_type;
} pci_dev_t;
```

## Enumeration

`pci_enumerate()` scans all PCI buses (0-255) and discovers devices:

1. For each bus/device/function, read vendor ID
2. If vendor ID is 0xFFFF, skip (no device)
3. Read class, subclass, prog_if, BARs, IRQ info
4. Store in `g_pci_devs[]` (up to 64 devices)

## BAR Decoding

For each device, the driver reads BAR0-BAR5 to determine:

- I/O port vs. memory-mapped
- Physical base address
- Region size (by writing all-1s and reading back)

## Usage

Other drivers query PCI devices after enumeration:

- AHCI driver looks for class 0x01 (mass storage), subclass 0x06 (SATA)
- VirtIO-net driver looks for vendor 0x1AF4 (VirtIO)
- UIO driver maps PCI BARs into user space for userspace drivers
