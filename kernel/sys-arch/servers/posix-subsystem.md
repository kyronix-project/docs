# POSIX Subsystem

This document describes the POSIX subsystem in Kyronix.

## Overview

The POSIX subsystem provides a compatibility layer between the Kyronix kernel and POSIX applications. It translates POSIX-style operations into the kernel's native interfaces.

## Functions

The POSIX subsystem handles:

- Process management (fork, exec, wait)
- Filesystem operations (open, read, write, close)
- Signal delivery
- Device access
- Terminal I/O

## Integration

In Kyronix, the kernel itself implements most POSIX syscalls directly (150+ Linux-compatible syscalls). The POSIX subsystem serves as an additional compatibility layer for operations that require user-space handling.

> **Note:** The POSIX subsystem is part of the Managarm-derived server infrastructure.
