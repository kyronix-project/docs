# Protocols

This document describes the protocols supported by the Kyronix kernel. It is the child of [Reference](index.md).

## Supported Protocols

- **Limine v3 boot protocol**: memory map, HHDM (Higher Half Direct Map), framebuffer, modules, RSDP (Root System Description Pointer), SMP (Symmetric Multi-Processing), kernel address.
- **PCI configuration**: standard I/O configuration space (ports 0xCF8/0xCFC).
- **ACPI** (Advanced Configuration and Power Interface): RSDP, RSDT/XSDT, MADT (Multiple APIC Description Table), FADT (Fixed ACPI Description Table).
- **AHCI** (Advanced Host Controller Interface): SATA (Serial ATA) host controller interface.
- **VirtIO 1.0**: paravirtualized network device.
- **lwIP TCP/IP** (lightweight IP): IPv4 (Internet Protocol version 4), TCP (Transmission Control Protocol), UDP (User Datagram Protocol), ICMP (Internet Control Message Protocol), ARP (Address Resolution Protocol).
- **PS/2**: keyboard and mouse input.
- **Syscall**: AMD64 SYSCALL/SYSRET mechanism.

Last reviewed: 2026-07-22
