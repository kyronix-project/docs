# VFS

This document describes the Virtual Filesystem Switch (VFS) in the Kyronix kernel. It is the child of [Filesystems](sys-arch/kernel/fs/index.md).

## Source

`kernel/fs/vfs.c`, `kernel/fs/vfs_internal.h`

## Overview

The VFS provides a unified interface for all filesystem operations. It manages mount points, file descriptor tables, node reference counting, and filesystem registration.

## Data Structures

### `vfs_node_t`

Represents a file, directory, device, or special node:

| Field | Description |
|---|---|
| `type` | Node type (VFS_TYPE_REG, VFS_TYPE_DIR, VFS_TYPE_DEV, etc.) |
| `name` | Node name |
| `size` | File size in bytes |
| `mode` | Permission bits |
| `uid`, `gid` | Owner and group |
| `data` | Filesystem-specific data |
| `fs_ops` | Filesystem operations (read, write, create, etc.) |
| `refcount` | Reference count |

### `vfs_file_t`

Represents an open file descriptor:

| Field | Description |
|---|---|
| `node` | Associated VFS node |
| `offset` | Current file position |
| `flags` | Open flags (O_RDONLY, O_WRONLY, etc.) |
| `pipe` | Pipe data (for pipe FDs) |
| `inet` | Inet socket data |

## Mount System

| Function | Description |
|---|---|
| `vfs_init()` | Initialize VFS, mount /proc, /sys, /dev/pts |
| `vfs_mount(path, fs, dev)` | Mount a filesystem at a path |
| `vfs_lookup(path)` | Look up a node by path |
| `vfs_register_fs(fs)` | Register a filesystem driver |

## Path Resolution

Paths are resolved relative to the process's current working directory (`cwd`) and jail root. The resolution process:

1. If path is absolute and jail root exists, prepend jail root
2. Walk components, resolving `.` and `..`
3. Return refcounted node pointer

## Filesystem Operations

Each registered filesystem provides:

| Operation | Description |
|---|---|
| `check_root(bd)` | Check if a block device contains this filesystem |
| `mount(bd, path)` | Mount the filesystem |
| `read(node, buf, off, len)` | Read data |
| `write(node, buf, off, len)` | Write data |
| `create(node, name, type)` | Create a file or directory |
| `unlink(node, name)` | Remove a file |
| `readdir(node, entries)` | List directory contents |

## Reference Counting

VFS nodes use reference counting to prevent use-after-free. `vfs_lookup()` increments the refcount; `vfs_node_unref_internal()` decrements it. When the refcount reaches zero, the node is freed.

Last reviewed: 2026-07-22
