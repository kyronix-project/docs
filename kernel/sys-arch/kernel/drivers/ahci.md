# AHCI

This document describes the AHCI (Advanced Host Controller Interface) driver in the Kyronix kernel.

## Overview

The AHCI driver provides access to SATA disk drives. It implements the AHCI specification for communicating with SATA controllers.

## Initialization

`ahci_init()` is called after PCI enumeration:

1. Find the AHCI controller (PCI class 0x01, subclass 0x06, prog_if 0x01)
2. Read the ABAR (AHCI Base Address Register) from BAR5
3. Map the AHCI MMIO registers into kernel space
4. Reset the controller (HBA reset)
5. Discover attached ports
6. Initialize port command lists and FIS structures
7. Register as a block device

## Block Device Interface

AHCI provides a block device interface for the VFS:

| Operation | Description |
|-----------|-------------|
| `block_read(dev, buf, sector, count)` | Read sectors from disk |
| `block_write(dev, buf, sector, count)` | Write sectors to disk |

## Integration

The AHCI driver works with the ext2 filesystem driver:

1. `pci_enumerate()` discovers the AHCI controller
2. `ahci_init()` sets up the block device
3. The ext2 driver checks the block device for a valid superblock
4. If found, the ext2 filesystem is mounted at `/`

## Limitations

- No Native Command Queuing (NCQ)
- No port multiplier support
- No hot-plug detection
