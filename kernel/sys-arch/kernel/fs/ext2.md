# ext2

This document describes the ext2 filesystem driver in the Kyronix kernel. It is the child of [Filesystems](sys-arch/kernel/fs/index.md).

## Source

`kernel/fs/ext2.c`

## Overview

The ext2 driver provides read/write access to ext2/3 filesystems. It is the primary disk-based filesystem used for the root partition.

## Features

- Block size: 4096 bytes
- Inode-based file storage
- Directory entries with inode references
- Symbolic link support
- File creation, deletion, renaming
- Permission bits (mode, uid, gid)

## Mount Process

1. Read superblock from block device
2. Validate magic number and block size
3. Initialize block and inode bitmaps
4. Mount at specified path

## Block Allocation

The driver maintains block and inode bitmaps. Free blocks are found via linear scan of the bitmap. Block groups are used to distribute metadata across the disk.

## Registration

```c
ext2_init();  // Register ext2 filesystem driver
```

After registration, the kernel attempts to mount ext2 on block devices during boot.

Last reviewed: 2026-07-22
