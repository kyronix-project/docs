# Hel

This document describes the Helio system call interface.

## Overview

Hel (Helio) provides an abstraction layer over the raw kernel syscalls. It is used by the Managarm-derived servers and drivers to interact with the kernel in a more structured way.

## Interface

Hel typically provides:

- Process management (create, terminate, wait)
- Memory management (map, unmap, protect)
- IPC primitives (send, receive, async operations)
- Handle management

> **Note:** Hel is a dependency inherited from the Managarm project. The Kyronix kernel itself provides a direct Linux-compatible syscall interface.
