# Filesystems

This document describes the filesystem subsystem of the Kyronix kernel. It is the child of [Kernel](sys-arch/kernel/index.md) and parent of filesystem component documents.

## Components

| Component | Source | Description |
|---|---|---|
| VFS | `fs/vfs.c` | Virtual Filesystem Switch layer |
| ext2 | `fs/ext2.c` | ext2/3 filesystem driver |
| FAT32 | `fs/fat32.c` | FAT32 filesystem driver |
| procfs | `fs/procfs.c` | Process information filesystem (/proc) |
| devfs | `fs/devfs.c` | Device filesystem (/dev) |
| eventfd | `fs/eventfd.c` | Event file descriptor |
| pipe | `fs/pipe.c` | Anonymous pipes |
| CPIO | `fs/cpio.c` | CPIO archive loader (initrd) |
| fstab | `fs/fstab.c` | /etc/fstab parser |
| Unix socket | `fs/unix_socket.c` | Unix domain sockets |
| Inet socket | `fs/inet_socket.c` | Internet domain sockets (lwIP) |

## Mount Points

The kernel mounts the following filesystems at boot:

| Mount | Source | Description |
|---|---|---|
| `/proc` | procfs | Process information |
| `/sys` | devfs | System/device nodes |
| `/dev/pts` | devfs | Pseudo-terminal devices |
| `/` | ext2 or initrd | Root filesystem |

## File Descriptor Operations

The VFS layer provides the following operations on file descriptors:

| Operation | Function |
|---|---|
| Read | `fd_read()` |
| Write | `fd_write()` |
| Seek | `fd_lseek()` |
| Close | `fd_close()` |
| Stat | `fd_stat()` / `fd_fstat()` |
| Dup | `fd_dup()` / `fd_dup2()` / `fd_dup3()` |
| Poll | `fd_pollin()` / `fd_pollout()` / `fd_pollhup()` |
| Ioctl | `fd_ioctl()` |
| Getdents | `fd_getdents64()` |

Last reviewed: 2026-07-22
