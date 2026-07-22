# CPIO

This document describes the CPIO archive loader in the Kyronix kernel. It is the child of [Filesystems](index.md).

## Source

`kernel/fs/cpio.c`
`kernel/fs/cpio.h`

## Overview

The CPIO loader provides read-only access to CPIO archives used as initial ramdisks (initrds). The loader extracts files from the CPIO archive passed by the Limine bootloader as a module.

## VFS Integration

The CPIO loader integrates with the VFS layer to present archive contents as a filesystem.

Last reviewed: 2026-07-22
