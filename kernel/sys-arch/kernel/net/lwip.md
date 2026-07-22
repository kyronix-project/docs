# lwIP

This document describes the lwIP integration in the Kyronix kernel. It is the child of [Networking](sys-arch/kernel/net/index.md).

## Source

`kernel/net/lwip/` (lwIP library), `kernel/net/lwip_glue.c`

## Overview

lwIP (Lightweight IP) is an open-source TCP/IP stack designed for embedded systems. Kyronix integrates lwIP with a glue layer that bridges its memory allocation to the kernel heap.

## Glue Layer

The `lwip_glue.c` file maps lwIP's expected libc functions to kernel implementations:

| lwIP Expects | Kernel Provides |
|---|---|
| `malloc(s)` | `kmalloc(s)` |
| `free(p)` | `kfree(p)` |
| `calloc(n, s)` | `kcalloc(n, s)` |
| `sys_now()` | `g_ticks * 10` (ticks to ms) |
| `strtol(s, end, base)` | Custom decimal parser |

## Protocols Supported

- TCP (connection-oriented)
- UDP (connectionless)
- ICMP (ping)
- ARP (Address Resolution Protocol)
- IPv4

## Initialization

```c
net_init();
```

1. Check `virtnet_ready()`
2. Call `lwip_init()`
3. Configure static IP (10.0.2.15/24, gateway 10.0.2.2)
4. Register kyronix netif with `ethernet_input` as input function
5. Set DNS server to 10.0.2.3

## Timeout Processing

lwIP timeouts are processed periodically via `sys_check_timeouts()`. This is called every 256 polls from `net_poll()`.

Last reviewed: 2026-07-22
