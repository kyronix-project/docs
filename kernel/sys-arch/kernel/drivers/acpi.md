# ACPI

This document describes the ACPI implementation in the Kyronix kernel. It is the child of [Drivers](sys-arch/kernel/drivers/index.md).

## Source

`kernel/drivers/acpi.c`

## Overview

ACPI (Advanced Configuration and Power Interface) provides hardware configuration information. The kernel parses the RSDP (Root System Description Pointer) to locate the ACPI tables.

## Initialization

```c
acpi_init(rsdp_address);
```

1. Read RSDP from the address provided by Limine
2. Validate RSDP signature ("RSD PTR ")
3. Locate RSDT/XSDT (Root/Extended System Description Table)
4. Parse SDT entries for MADT (APIC), FADT (Fixed ACPI), and others

## Key Tables

| Table | Purpose |
|---|---|
| RSDP | Root pointer to all ACPI tables |
| RSDT/XSDT | Root/Extended System Description Table |
| MADT | Multiple APIC Description Table (CPU topology) |
| FADT | Fixed ACPI Description Table (PM timer, reset port) |

## Functions

| Function | Description |
|---|---|
| `acpi_init(addr)` | Parse ACPI tables from RSDP address |
| `acpi_available()` | Check if ACPI was successfully initialized |

Last reviewed: 2026-07-22
