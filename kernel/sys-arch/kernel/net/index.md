# Networking

This section describes the networking subsystem of the Kyronix kernel.

## Components

- [lwIP](lwip.md) -- lightweight IP stack integration
- [Network Interface](netif.md) -- VirtIO-net adapter

## Architecture

Kyronix uses lwIP (lightweight IP) as its TCP/IP networking stack. The network stack is integrated with the VirtIO-net paravirtualized network device for use in QEMU/KVM environments.

```
+-------------------+
|  User Applications|
+-------------------+
|  Socket Layer     |  (inet_socket.c / unix_socket.c)
+-------------------+
|  lwIP Stack       |  (tcpip, ip, icmp, udp, tcp)
+-------------------+
|  kyronix_netif    |  ( VirtIO-net adapter)
+-------------------+
|  VirtIO-net       |  (paravirtualized NIC)
+-------------------+
```

## Initialization

1. `virtnet_init()` discovers and initializes the VirtIO-net device
2. `net_init()` initializes lwIP and registers the kyronix_netif
3. The network interface obtains an IP address via DHCP or static configuration
