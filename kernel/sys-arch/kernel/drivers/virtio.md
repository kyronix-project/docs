# VirtIO

This document describes the VirtIO network driver in the Kyronix kernel. It is the child of [Drivers](sys-arch/kernel/drivers/index.md).

## Source

`kernel/drivers/virtio_net.c`

## Overview

VirtIO provides paravirtualized network access for QEMU virtual machines. The driver implements the VirtIO 1.0 network device specification.

## Initialization

1. Scan PCI devices for VirtIO vendor ID (`0x1AF4`)
2. Negotiate feature bits with the device
3. Initialize virtqueues (TX and RX)
4. Register MAC address (6 bytes)
5. Set link status to up

## Virtqueue Layout

| Queue | Purpose |
|---|---|
| TX queue | Transmit Ethernet frames |
| RX queue | Receive Ethernet frames |

## Functions

| Function | Description |
|---|---|
| `virtnet_init()` | Initialize VirtIO-net device |
| `virtnet_ready()` | Check if device is operational |
| `virtnet_send(buf, len)` | Transmit an Ethernet frame |
| `virtnet_recv(buf, len)` | Receive an Ethernet frame |
| `virtnet_mac()` | Get MAC address |

## Network Integration

The VirtIO-net driver is connected to the lwIP network stack via the kyronix netif layer. Raw Ethernet frames are passed between the driver and lwIP without additional encapsulation.

Last reviewed: 2026-07-22
