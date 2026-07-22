# PCI Enumeration

This document describes the PCI bus enumeration implementation in the Kyronix kernel. It is the child of [Driver Implementation Notes](index.md).

The Kyronix kernel enumerates the PCI bus using Port I/O-based configuration space access to discover and catalog all connected hardware devices.

## Configuration Space Access

PCI configuration space is accessed through two standard I/O ports:

- **0xCF8 (Address register):** Receives the configuration address, which encodes the bus number, device number, function number, and register offset.
- **0xCFC (Data register):** Provides read/write access to the configuration data at the specified address.

## Enumeration Procedure

1. Iterate over all 256 buses.
2. For each bus, iterate over all 32 devices.
3. For each device, iterate over all 8 functions.
4. Read the Vendor ID register at offset 0x00.
5. If the Vendor ID returns `0xFFFF`, the device or function is absent; skip to the next function.
6. Read the Class Code, Subclass, Programming Interface (prog_if), Base Address Register (BAR) registers, and Header Type from the configuration space.
7. Store the device information in the global device table.

## Usage

The global device table populated by PCI enumeration is consumed by the following drivers to discover and initialize their respective hardware:

- AHCI (Advanced Host Controller Interface) driver
- VirtIO driver
- Other PCI device drivers that query the device table during their probe sequences

Last reviewed: 2026-07-22
