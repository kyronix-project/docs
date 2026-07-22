# procfs

This document describes the procfs filesystem in the Kyronix kernel. It is the child of [Filesystems](sys-arch/kernel/fs/index.md).

## Source

`kernel/fs/procfs.c`

## Overview

procfs is an in-memory filesystem mounted at `/proc` that exposes process and kernel information. It provides a read-only view of system state.

## Mount Point

`/proc`

## Entries

| Entry | Description |
|---|---|
| `/proc/<pid>/` | Per-process information directories |
| `/proc/self/` | Symlink to current process |
| `/proc/meminfo` | Memory usage statistics |
| `/proc/version` | Kernel version string |
| `/proc/uptime` | System uptime |

## Implementation

procfs nodes are dynamically generated on lookup. The filesystem does not persist data to disk; all information is gathered from kernel state at access time.

Last reviewed: 2026-07-22
