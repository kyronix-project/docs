# procfs

This document describes the procfs virtual filesystem in the Kyronix kernel.

## Overview

procfs provides a virtual filesystem at `/proc` that exposes kernel and process information as files. It implements a subset of the Linux `/proc` interface.

## Mounted Files

| Path | Description |
|------|-------------|
| `/proc/meminfo` | Memory usage information |
| `/proc/version` | Kernel version string |
| `/proc/uptime` | System uptime |
| `/proc/self` | Symlink to the current process's `/proc/<pid>` directory |
| `/proc/<pid>/` | Per-process information |

## Per-Process Files

| Path | Description |
|------|-------------|
| `/proc/<pid>/status` | Process status (name, state, PID, PPID, etc.) |
| `/proc/<pid>/maps` | Memory maps (VMA regions) |
| `/proc/<pid>/fd/` | File descriptor symlinks |

## Implementation

procfs is implemented as an in-memory filesystem. Files are generated dynamically on read. The filesystem driver registers via `vfs_register_fs()` and creates nodes under `/proc` during `vfs_init()`.

## Integration

procfs is mounted automatically during kernel initialization. The mount is performed in `vfs_init()` before user-space processes are started.
