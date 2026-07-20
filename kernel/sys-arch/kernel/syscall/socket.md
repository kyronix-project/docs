# Socket Operations

This document describes the socket-related syscalls in the Kyronix kernel.

## Syscalls

| Syscall | Number | Description |
|---------|--------|-------------|
| `socket` | 41 | Create a socket |
| `connect` | 42 | Initiate a connection |
| `accept` | 43 | Accept a connection |
| `sendto` | 44 | Send a message |
| `recvfrom` | 45 | Receive a message |
| `sendmsg` | 46 | Send a message with ancillary data |
| `recvmsg` | 47 | Receive a message with ancillary data |
| `shutdown` | 48 | Shut down a socket |
| `bind` | 49 | Bind a socket to an address |
| `listen` | 50 | Listen for connections |
| `getsockname` | 51 | Get socket name |
| `getpeername` | 52 | Get peer name |
| `setsockopt` | 54 | Set socket option |
| `getsockopt` | 55 | Get socket option |
| `socketpair` | 53 | Create a pair of connected sockets |

## Socket Types

### Unix Domain Sockets

- `AF_UNIX` / `AF_LOCAL`
- `SOCK_STREAM` (reliable, ordered)
- `SOCK_DGRAM` (unreliable, unordered)

Features:

- `SCM_RIGHTS` -- file descriptor passing via ancillary data
- `SCM_CREDENTIALS` / `SO_PEERCRED` -- peer credential retrieval
- `SO_PASSCRED` -- receive credentials with each message
- Path binding, listen, accept, connect

### Inet Sockets

- `AF_INET` (IPv4)
- `SOCK_STREAM` (TCP)
- `SOCK_DGRAM` (UDP)

Features:

- TCP connections via lwIP
- UDP send/receive
- `bind`, `listen`, `accept`, `connect`
- `sendmsg`/`recvmsg` with scatter-gather I/O

## Ancillary Data

`sendmsg`/`recvmsg` support ancillary data (control messages):

| Type | Description |
|------|-------------|
| `SCM_RIGHTS` | File descriptor passing |
| `SCM_CREDENTIALS` | Process credentials |

## Socket Options

Supported socket options:

| Option | Level | Description |
|--------|-------|-------------|
| `SO_REUSEADDR` | SOL_SOCKET | Allow address reuse |
| `SO_KEEPALIVE` | SOL_SOCKET | Enable TCP keepalive |
| `SO_RCVBUF` | SOL_SOCKET | Receive buffer size |
| `SO_SNDBUF` | SOL_SOCKET | Send buffer size |

## Poll Support

Sockets support `poll`/`select`/`epoll`:

- `fd_pollin()` -- data available for reading
- `fd_pollout()` -- can write without blocking
- `fd_pollhup()` -- connection closed
