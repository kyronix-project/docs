# libblockfs

This document describes the libblockfs block device filesystem library.

## Overview

libblockfs is a user-space library for accessing block devices. It provides a filesystem abstraction over raw block device I/O, enabling user-space filesystem implementations.

## Functions

- Open and read block device partitions
- Provide filesystem-like access to block device contents
- Cache block reads for performance

## Integration

libblockfs is used by the Managarm-derived servers for disk-backed filesystem access. In Kyronix, the kernel handles block device access directly via the AHCI driver and ext2 filesystem.

> **Note:** libblockfs is a dependency inherited from the Managarm project.
