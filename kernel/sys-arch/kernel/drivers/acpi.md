# ACPI

This document describes the ACPI (Advanced Configuration and Power Interface) driver in the Kyronix kernel.

## Overview

The ACPI driver provides power management and system control through the ACPI tables provided by the firmware.

## Initialization

`acpi_init(rsdp_phys)` is called during boot with the physical address of the RSDP (Root System Description Pointer), obtained from the Limine `LIMINE_RSDP_REQUEST`.

## Functions

| Function | Description |
|----------|-------------|
| `acpi_init(rsdp_phys)` | Parse ACPI tables from RSDP |
| `acpi_available()` | Check if ACPI is initialized |
| `acpi_poweroff()` | Power off the system |
| `acpi_reboot()` | Reboot the system |

## RSDP Discovery

The RSDP pointer is provided by the Limine bootloader via the `LIMINE_RSDP_REQUEST`. For base revision >= 3, this is a physical address.

## Power Management

The ACPI driver provides system-level power control:

- `acpi_poweroff()` -- ACPI shutdown (writes to PM1a/PM1b control blocks)
- `acpi_reboot()` -- ACPI reset (writes to reset register)

These functions are called from the kernel when the user requests a shutdown or reboot.
