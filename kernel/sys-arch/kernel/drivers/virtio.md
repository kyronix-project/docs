# VirtIO

This document describes the VirtIO device support in the Kyronix kernel.

## Overview

VirtIO provides paravirtualized device access for virtual machines. Kyronix implements VirtIO-net for network connectivity in QEMU/KVM environments.

## VirtIO-net

The VirtIO network driver provides a virtual network interface for the kernel.

### Initialization

`virtnet_init()` is called during boot:

1. Find the VirtIO-net device (PCI vendor 0x1AF4, device 0x1000)
2. Negotiate features with the device
3. Set up virtqueues (receive and transmit)
4. Map device MMIO registers
5. Register with the lwIP networking stack

### Virtqueues

The driver uses two virtqueues:

- **Receive queue:** Device places incoming packets here
- **Transmit queue:** Driver places outgoing packets here

Each virtqueue uses a ring buffer of descriptors, available/used rings, and notification mechanisms.

### Integration

VirtIO-net integrates with the lwIP networking stack via the `kyronix_netif` adapter:

1. `virtnet_init()` creates the network interface
2. `net_init()` initializes lwIP and registers the interface
3. Incoming packets are passed to lwIP via `netif_input()`
4. Outgoing packets are sent via the transmit virtqueue
