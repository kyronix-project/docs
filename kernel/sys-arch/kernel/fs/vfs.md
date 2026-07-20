# Virtual Filesystem (VFS)

This document describes the Virtual Filesystem layer in the Kyronix kernel.

## Overview

The VFS provides a unified interface for all filesystem operations, abstracting away the differences between disk-backed filesystems (ext2, FAT32), virtual filesystems (procfs, devfs), and IPC primitives (pipes, sockets).

## Node Structure

Every file, directory, device, or socket is represented by a `vfs_node_t`:

```c
typedef struct vfs_node {
    char name[256];
    uint8_t type;         // VFS_TYPE_REG, VFS_TYPE_DIR, VFS_TYPE_CHR, etc.
    uint32_t mode;        // POSIX permission bits
    uint32_t uid, gid;    // ownership
    uint32_t ino;         // inode number
    uint32_t refcnt;      // reference count
    uint8_t deleted;      // marked for deletion
    uint64_t size;        // file size in bytes
    uint8_t *data;        // file data (ramfs/chr)
    uint64_t capacity;    // allocated capacity
    struct vfs_node *children;  // child linked list (directories)
    struct vfs_node *next;      // sibling link
    struct vfs_node *parent;    // parent directory
    char *symlink;        // symlink target
    // character device operations
    int64_t (*chr_read)(...);
    int64_t (*chr_write)(...);
    int64_t (*chr_ioctl)(...);
    bool (*chr_pollin)(...);
    // filesystem operations
    void *fs_private;     // per-node filesystem data (e.g., ext2 inode)
    struct vfs_fs_ops *fs_ops;  // filesystem operations
} vfs_node_t;
```

## File Descriptor Structure

Each open file descriptor is represented by a `vfs_file_t`:

```c
typedef struct {
    uint64_t magic;
    vfs_node_t *node;     // associated node (NULL for pipes/sockets)
    uint64_t pos;         // current file position
    int flags;            // open flags (O_RDONLY, O_WRONLY, etc.)
    pipe_t *pipe;         // pipe data (for pipe FDs)
    int pipe_end;         // pipe end (0=read, 1=write)
    // socket fields
    struct net_conn *inet; // inet socket connection
    // eventfd/timerfd
    eventfd_state_t *efd;
    timerfd_state_t *tfd;
    uint8_t cloexec;      // close-on-exec flag
} vfs_file_t;
```

## Filesystem Driver Registration

Disk-backed filesystems register via `vfs_register_fs()`:

```c
struct filesystem {
    const char *name;
    bool (*check_root)(struct block_device *);
    bool (*mount)(struct block_device *, const char *);
    int (*sync)(void);
    int (*create)(struct vfs_node *node, const char *path, uint32_t mode);
};
```

## Path Resolution

- `vfs_lookup(path)` -- resolve a path to a `vfs_node_t`
- `vfs_lookup_nofollow(path)` -- resolve without following the final symlink
- `at_resolve(dirfd, path, out, sz)` -- resolve relative to a directory FD (supports `AT_FDCWD`)

## Initialization

`vfs_init()` creates the initial in-memory filesystem tree:

1. Create root directory `/`
2. Create `/dev`, `/proc`, `/sys` directories
3. Mount procfs at `/proc`
4. Mount devfs at `/dev`
5. Create `/dev/pts` for pseudo-terminals

## Limits

| Resource | Limit |
|----------|-------|
| Max open FDs | 1024 per process |
| Max node name | 256 characters |
| Max path | 512 characters |
