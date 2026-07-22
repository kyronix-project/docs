# AHCI

This document describes the AHCI (Advanced Host Controller Interface) driver in the Kyronix kernel. It is the child of [Drivers](sys-arch/kernel/drivers/index.md).

## Source

`kernel/drivers/ahci.c`

## Overview

AHCI provides access to SATA storage devices. The driver initializes AHCI controllers discovered via PCI enumeration and registers block devices for each attached drive.

## Initialization

1. Scan PCI devices for AHCI class (class=0x01, subclass=0x06)
2. Read ABAR (AHCI Base Address Register) from PCI BAR5
3. Map AHCI HBA registers into kernel virtual space
4. Reset the HBA and detect attached ports
5. For each active port, initialize command list and FIS structures
6. Register as block device via `block_register()`

## Block Device Interface

AHCI block devices are accessed through the generic block device layer:

```c
struct block_device {
    void (*read)(uint64_t lba, uint32_t count, void *buf);
    void (*write)(uint64_t lba, uint32_t count, const void *buf);
    uint64_t total_sectors;
    // ...
};
```

## Functions

| Function | Description |
|---|---|
| `ahci_init()` | Initialize AHCI controller(s) |
| `ahci_ready()` | Check if AHCI is operational |

NOTE: AHCI devices appear as `/dev/ahci0` via devfs.

Last reviewed: 2026-07-22
