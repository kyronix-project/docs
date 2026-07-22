# FAT32

This document describes the FAT32 filesystem driver in the Kyronix kernel. It is the child of [Filesystems](index.md).

## Source

`kernel/fs/fat32.c`
`kernel/fs/fat32.h`

## Overview

The FAT32 driver provides read/write access to FAT32-formatted disk partitions. The driver supports directory traversal, file read/write, and file creation.

## Features

- Directory traversal
- File read/write
- File creation

## VFS Integration

The FAT32 driver integrates with the VFS layer through standard filesystem operations.

Last reviewed: 2026-07-22
