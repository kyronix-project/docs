# PCI

This document describes the PCI bus enumeration in the Kyronix kernel. It is the child of [Drivers](sys-arch/kernel/drivers/index.md).

## Source

`kernel/drivers/pci.c`

## Overview

PCI (Peripheral Component Interconnect) enumeration discovers and configures all PCI devices on the bus. The kernel scans the standard I/O configuration space to identify devices.

## Configuration Space Access

PCI configuration is accessed via I/O ports:

| Port | Description |
|---|---|
| `0xCF8` | Configuration Address Register |
| `0xCFC` | Configuration Data Register |

## Device Structure

```c
struct pci_device {
    uint8_t  bus, dev, func;
    uint16_t vendor_id, device_id;
    uint8_t  class, subclass;
    uint8_t  prog_if;
    uint8_t  header_type;
    uint32_t bar[6];
    uint16_t command;
};
```

## Enumeration

`pci_enumerate()` scans all 256 buses, 32 devices, and 8 functions:

1. For each device/function, read vendor ID
2. If vendor ID is `0xFFFF`, device does not exist -- skip
3. Read class, subclass, BAR registers, and header type
4. Store in global device table

## BAR (Base Address Register) Types

| Type | Description |
|---|---|
| I/O | Memory-mapped I/O port |
| Memory | Memory-mapped device registers |

BARs are used by drivers (AHCI, VirtIO) to access device registers.

Last reviewed: 2026-07-22
