# Socket Operations

This document describes the socket operation syscalls in the Kyronix kernel. It is the child of [Syscalls](index.md).

## Syscall Table

- 41 `socket` - Create a socket endpoint
- 42 `connect` - Initiate a connection on a socket
- 43 `accept` - Accept a connection on a socket
- 44 `sendto` - Send a message on a socket
- 45 `recvfrom` - Receive a message from a socket
- 46 `sendmsg` - Send a message with ancillary data
- 47 `recvmsg` - Receive a message with ancillary data
- 48 `shutdown` - Shut down part of a full-duplex connection
- 49 `bind` - Bind a socket to an address
- 50 `listen` - Listen for connections on a socket
- 51 `getsockname` - Get local address of a socket
- 52 `getpeername` - Get remote address of a socket
- 53 `socketpair` - Create a pair of connected sockets
- 54 `setsockopt` - Set a socket option
- 55 `getsockopt` - Get a socket option
- 288 `accept4` - Accept a connection with flags

## Socket Types

The `socket` syscall routes to one of two backends based on the domain argument.

### AF_UNIX (Unix Domain Sockets)

Unix domain sockets are implemented in `unix_socket.c`. The domain value is 1. Only `SOCK_STREAM` is supported. Unix domain sockets provide local inter-process communication (IPC) using pipe-backed data transfer.

The socket lifecycle follows these states:

1. `SOCK_UNBOUND` - Socket created but not yet bound to a path
2. `SOCK_BOUND` - Socket bound to a filesystem path or abstract name
3. `SOCK_LISTENING` - Socket in listening state, accepting connections

Abstract sockets use names prefixed with a null byte. The kernel maintains a table of `MAX_ABSTRACT_SOCKS` (16) entries for abstract socket names. Abstract sockets are invisible across jails when the `JAILF_IPC` flag is set on the originating jail.

### AF_INET (Internet Sockets)

Internet sockets are implemented in `inet_socket.c` using the lwIP (Lightweight IP) stack. The domain value is 2. Three socket types are supported:

- `SOCK_STREAM` (1) - Transmission Control Protocol (TCP) connections backed by `tcp_pcb`
- `SOCK_DGRAM` (2) - User Datagram Protocol (UDP) connections backed by `upcb`
- `SOCK_RAW` (3) - Raw IP connections backed by `raw_pcb`

Each internet socket contains a `net_conn_t` structure with protocol control blocks, a 16 KiB receive ring buffer for TCP, and an 8-slot datagram queue for UDP/raw sockets.

## SCM_RIGHTS (File Descriptor Passing)

The `SCM_RIGHTS` ancillary data type enables file descriptor passing between processes over Unix domain sockets. The constant value is 1.

1. The sender calls `sendmsg` with a control message containing `SOL_SOCKET`, `SCM_RIGHTS`, and an array of file descriptors
2. The kernel extracts VFS (Virtual File System) node pointers from the sender's file descriptors via `fd_get_node()`
3. Nodes are queued into the pipe's ancillary data ring via `pipe_anc_send()`
4. The receiver calls `recvmsg` and the kernel reconstructs file descriptors from the queued VFS nodes via `fd_open_node()`
5. A maximum of `PIPE_ANC_MAXFDS` file descriptors are transferred per message

## SCM_CREDENTIALS (Peer Credential Passing)

The `SCM_CREDENTIALS` ancillary data type enables peer credential passing. The constant value is 2. The `SO_PASSCRED` socket option (value 16) must be enabled on the receiving socket to receive credentials.

1. The receiver enables credential passing via `setsockopt(SOL_SOCKET, SO_PASSCRED, &on)`
2. The receiver calls `recvmsg` with a control buffer large enough to hold a `ucred_s` structure
3. The kernel fills the control message with the sender's process ID (PID), user ID (UID), and group ID (GID)
4. The `ucred_s` structure contains `pid` (int32), `uid` (uint32), and `gid` (uint32)

## Socket Options

Socket options are managed via `setsockopt` (syscall 54) and `getsockopt` (syscall 55) at the `SOL_SOCKET` (level 1) layer.

### Supported Options

| Option | Value | Direction | Description |
|---|---|---|---|
| `SO_TYPE` | 3 | Get | Returns socket type (1=stream, 2=dgram, 3=raw) |
| `SO_ERROR` | 4 | Get | Returns last error code (always 0) |
| `SO_PASSCRED` | 16 | Set | Enables `SCM_CREDENTIALS` delivery |
| `SO_PEERCRED` | 17 | Get | Returns peer PID/UID/GID in `ucred_s` |
| `SO_DOMAIN` | 39 | Get | Returns address family (1=AF_UNIX, 2=AF_INET) |

## sendmsg / recvmsg (Vectored I/O)

The `sendmsg` (syscall 46) and `recvmsg` (syscall 47) syscalls support vectored I/O through scatter-gather lists of `iovec` structures.

The `msghdr` layout in memory:

| Offset | Field | Type |
|---|---|---|
| 0 | msg_name | void pointer |
| 8 | msg_namelen | uint32 |
| 12 | msg_iov | iovec pointer |
| 16 | msg_iovlen | int |
| 24 | msg_control | void pointer |
| 32 | msg_controllen | uint64 |
| 40 | msg_flags | uint32 |

### sendmsg

1. Validate the `msghdr` pointer and `iovec` array
2. For each iovec entry, call `fd_write()` with the iovec base and length
3. If a control buffer is present, parse `cmsg_len`, `cmsg_level`, and `cmsg_type`
4. For `SCM_RIGHTS` messages, extract file descriptors and pass their VFS nodes through the pipe's ancillary ring

### recvmsg

1. For internet sockets, delegate to `inet_recvfrom()` for the first iovec entry and fill `msg_name` with the source address
2. For Unix domain sockets, read data from iovec entries via `fd_read()` or `fd_peek()` (when `MSG_PEEK` is set)
3. If ancillary data is available in the pipe's receive ring, construct an `SCM_RIGHTS` control message with reconstructed file descriptors
4. If `SO_PASSCRED` is enabled and credentials are available, construct an `SCM_CREDENTIALS` control message with peer PID/UID/GID

Last reviewed: 2026-07-22
