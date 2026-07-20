# Filesystems

This section describes the filesystem subsystem of the Kyronix kernel.

## Components

- [Virtual Filesystem (VFS)](vfs.md) -- unified filesystem interface
- [ext2](ext2.md) -- ext2/EXT2 disk filesystem
- [procfs](procfs.md) -- /proc virtual filesystem
- [devfs](devfs.md) -- /dev virtual filesystem
- [eventfd](eventfd.md) -- event file descriptors
- [pipe](pipe.md) -- pipe and FIFO implementation

## Supported Filesystem Types

| Type | Description |
|------|-------------|
| Regular file | Disk-backed files (ext2, FAT32, CPIO) |
| Directory | Container for other nodes |
| Symlink | Symbolic link |
| Character device | Device nodes (/dev/null, /dev/tty, etc.) |
| Socket | Unix domain and inet sockets |
| Pipe | Unnamed and named pipes (FIFO) |

## Filesystem Drivers

Registered filesystem drivers:

| Driver | Description |
|--------|-------------|
| `ext2` | ext2/EXT2 read/write filesystem |
| `fat32` | FAT32 filesystem (partial) |
| `cpio` | CPIO initrd filesystem (read-only) |
| `procfs` | /proc virtual filesystem |
| `devfs` | /dev virtual filesystem |

## Mount Points

At boot, the kernel mounts:

- `/proc` -- procfs
- `/sys` -- sysfs (placeholder)
- `/dev/pts` -- devpts (pseudo-terminal slaves)
- `/` -- root filesystem (ext2 from disk or CPIO initrd)

## File Descriptor Operations

The VFS provides a unified interface for file operations:

| Operation | Description |
|-----------|-------------|
| `fd_open` / `fd_openat` | Open a file |
| `fd_close` | Close a file |
| `fd_read` / `fd_pread` | Read from a file |
| `fd_write` / `fd_pwrite` | Write to a file |
| `fd_lseek` | Seek within a file |
| `fd_stat` / `fd_fstat` | Get file status |
| `fd_getdents64` | Read directory entries |
| `fd_ioctl` | Device-specific control |
| `fd_fcntl` | File control |
| `fd_dup` / `fd_dup2` / `fd_dup3` | Duplicate file descriptors |
| `fd_readlink` | Read symlink target |
| `fd_pipe` / `fd_socketpair` | Create pipe/socket pairs |
| `fd_eventfd` | Create eventfd |
| `fd_timerfd_create` | Create timerfd |
