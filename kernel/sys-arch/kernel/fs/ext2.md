# ext2

This document describes the ext2 filesystem implementation in the Kyronix kernel.

## Overview

Kyronix includes an ext2 filesystem driver for reading and writing ext2-formatted disk partitions. ext2 is used as the primary disk-backed filesystem for the root partition.

## Features

- Read and write support for regular files and directories
- Symlink support
- Permission bits (mode, uid, gid)
- File size up to the ext2 limit
- Block-based allocation

## Integration

The ext2 driver registers as a filesystem driver via `vfs_register_fs()`. During boot:

1. `pci_enumerate()` discovers block devices
2. `ahci_init()` initializes SATA/AHCI controllers
3. The ext2 driver checks each block device for a valid ext2 superblock
4. If found, it mounts the filesystem and populates the VFS tree

## Disk I/O

ext2 reads and writes are performed through the block device layer, which provides:

- `block_read(dev, buf, sector, count)` -- read sectors
- `block_write(dev, buf, sector, count)` -- write sectors
- Sector size: 512 bytes (standard)

## Limitations

- No journaling (ext2, not ext3/ext4)
- No extended attributes
- No per-directory index
- Read-write support is functional but may not handle all edge cases

> **Note:** For the initial boot, the root filesystem can also be provided as a CPIO initrd, bypassing ext2 entirely.
