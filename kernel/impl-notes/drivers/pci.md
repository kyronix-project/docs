# PCI Enumeration

This document describes the PCI enumeration implementation.

## Configuration Mechanism

The PCI configuration space is accessed via the standard x86 mechanism:

- **Address port:** 0xCF8 (32-bit register)
- **Data port:** 0xCFC (32-bit register)

### Address Format

```
Bit 31:    Enable bit
Bits 30-24: Reserved
Bits 23-16: Bus number
Bits 15-11: Device number
Bits 10-8:  Function number
Bits 7-2:   Register number ( dword-aligned)
Bits 1-0:   Always 0
```

## Scanning Algorithm

`pci_enumerate()` scans all 256 buses, 32 devices, 8 functions:

1. For each bus/device/function, read vendor ID at offset 0
2. If vendor ID is 0xFFFF, no device exists (skip)
3. Read class code (offset 8, bits 23-16), subclass (bits 15-8), prog_if (bits 7-0)
4. Read BAR0-BAR5 to determine MMIO regions and sizes
5. Read IRQ line/pin from header
6. Store in `g_pci_devs[]` (up to 64 devices)

## BAR Decoding

To determine BAR size:

1. Write all-1s to the BAR register
2. Read back the value
3. The size is encoded in the zero bits (memory BAR) or the low 2 bits (I/O BAR)

## Device Classes

| Class | Subclass | Description |
|-------|----------|-------------|
| 0x01 | 0x06 | SATA/AHCI controller |
| 0x02 | 0x00 | Ethernet controller (VirtIO-net) |
| 0x06 | 0x04 | PCI-to-PCI bridge |
| 0xFF | 0xFF | UIO device (custom) |
