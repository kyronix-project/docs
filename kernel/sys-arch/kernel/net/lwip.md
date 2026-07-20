# lwIP

This document describes the lwIP integration in the Kyronix kernel.

## Overview

lwIP (lightweight IP) is a lightweight TCP/IP stack designed for embedded systems. Kyronix uses it to provide TCP/IP networking capabilities.

## Configuration

The lwIP configuration is defined in `lwipopts.h`:

| Option | Value | Description |
|--------|-------|-------------|
| `NO_SYS` | 0 | Operating system mode (with threads) |
| `LWIP_NETCONN` | 1 | Netconn API support |
| `LWIP_SOCKET` | 1 | Socket API support |
| `LWIP_DHCP` | 1 | DHCP client |
| `LWIP_UDP` | 1 | UDP support |
| `LWIP_TCP` | 1 | TCP support |
| `LWIP_ICMP` | 1 | ICMP (ping) support |

## OS Glue Layer

lwIP requires an OS abstraction layer for:

- Semaphores (`sys_sem_new`, `sys_sem_signal`, `sys_sem_wait`)
- Mailboxes (`sys_mbox_new`, `sys_mbox_post`, `sys_mbox_fetch`)
- Threads (`sys_thread_new`)
- Timeouts (`sys_timeouts_sleeptime`)

These are implemented in `lwip_glue.c` using Kyronix's synchronization primitives.

## Integration

The lwIP stack runs in its own thread context (`tcpip_thread`). Network packets flow through:

1. VirtIO-net receives a packet
2. kyronix_netif passes it to lwIP via `netif_input()`
3. lwIP processes the packet (IP, TCP, UDP, ICMP)
4. Data is delivered to the socket layer
5. Outgoing data follows the reverse path
