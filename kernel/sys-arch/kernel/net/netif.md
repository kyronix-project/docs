# Network Interface

This document describes the kyronix_netif adapter that connects VirtIO-net to lwIP.

## Overview

The kyronix_netif acts as a bridge between the VirtIO-net hardware driver and the lwIP networking stack. It implements the lwIP `netif` interface.

## Packet Flow

### Receive Path

1. VirtIO-net interrupt handler reads a packet from the receive virtqueue
2. The packet is passed to kyronix_netif
3. kyronix_netif calls `netif_input()` to deliver to lwIP
4. lwIP processes the packet through the IP/TCP/UDP stack

### Transmit Path

1. lwIP calls `kyronix_netif_output()` to send a packet
2. kyronix_netif places the packet in the VirtIO-net transmit virtqueue
3. VirtIO-net transmits the packet on the wire

## Network Configuration

The network interface can be configured via:

- DHCP (automatic IP address assignment)
- Static configuration (manual IP, netmask, gateway)

## Interface Parameters

| Parameter | Description |
|-----------|-------------|
| IP address | IPv4 address of the interface |
| Netmask | Subnet mask |
| Gateway | Default gateway address |
| MAC address | Hardware address from VirtIO-net |
