# Networking

This document describes the networking subsystem of the Kyronix kernel. It is the child of [Kernel](sys-arch/kernel/index.md) and parent of networking component documents.

## Components

| Component | Source | Description |
|---|---|---|
| Network Stack | `net/net.c` | lwIP initialization and polling |
| lwIP Glue | `net/lwip_glue.c` | Kernel memory allocator bridge |
| Kyronix Netif | `net/netif/kyronix_netif.c` | lwIP network interface driver |
| VirtIO-Net | `drivers/virtio_net.c` | Hardware NIC driver |

## Architecture

```
User-space (sockets)
    |
    v
Syscall layer (socket.c)
    |
    v
lwIP TCP/IP stack
    |
    v
kyronix_netif (lwIP netif)
    |
    v
virtio-net driver
    |
    v
QEMU virtio-net device
```

## Network Configuration

| Setting | Value |
|---|---|
| IP Address | 10.0.2.15 |
| Subnet Mask | 255.255.255.0 (/24) |
| Gateway | 10.0.2.2 |
| DNS Server | 10.0.2.3 |

These are static addresses matching QEMU's default user-mode networking (SLIRP) configuration.

## Socket Types

| Type | Source | Description |
|---|---|---|
| Unix domain | `fs/unix_socket.c` | Local IPC sockets |
| Internet | `fs/inet_socket.c` | TCP/UDP over lwIP |

## Functions

| Function | Description |
|---|---|
| `net_init()` | Initialize lwIP and register network interface |
| `net_poll()` | Drain virtio-net RX queue, process lwIP timeouts |
| `net_receive(frame, len)` | Feed raw Ethernet frame to lwIP |

Last reviewed: 2026-07-22
