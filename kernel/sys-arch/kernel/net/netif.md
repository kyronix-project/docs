# Network Interface

This document describes the Kyronix network interface driver in the Kyronix kernel. It is the child of [Networking](sys-arch/kernel/net/index.md).

## Source

`kernel/net/netif/kyronix_netif.c`

## Overview

The kyronix netif bridges lwIP to the virtio-net hardware. It implements the lwIP `netif` interface for sending and receiving raw Ethernet frames.

## Netif Configuration

| Property | Value |
|---|---|
| Name | `"e0"` |
| MTU | 1500 |
| Output (ARP+IP) | `etharp_output` |
| Link output | `kyronix_netif_output` |
| Flags | `NETIF_FLAG_BROADCAST \| NETIF_FLAG_ETHARP \| NETIF_FLAG_LINK_UP \| NETIF_FLAG_UP` |
| MAC address | From `virtnet_mac()` |

## Functions

| Function | Description |
|---|---|
| `kyronix_netif_init(nif)` | Initialize netif (called by `netif_add`) |
| `kyronix_netif_input(nif, data, len)` | Feed raw Ethernet frame into lwIP |
| `kyronix_netif_output(nif, p)` | Send pbuf chain via virtio-net |

## Receive Path

1. virtio-net driver calls `net_receive(frame, len)`
2. `net_receive` calls `kyronix_netif_input()`
3. `kyronix_netif_input` allocates a `pbuf` from `PBUF_POOL`
4. Copies frame data into pbuf chain
5. Calls `nif->input()` (which is `ethernet_input`)

## Transmit Path

1. lwIP calls `kyronix_netif_output(nif, p)`
2. Gathers pbuf chain into flat buffer (max 1514 bytes)
3. Calls `virtnet_send(buf, total)`

Last reviewed: 2026-07-22
