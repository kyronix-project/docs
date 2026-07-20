# udev

This document describes the udev device manager in Kyronix.

## Overview

udev manages device nodes in `/dev`. It monitors kernel device events and creates/removes device nodes accordingly.

## Functions

- Receives device add/remove events from the kernel
- Creates device nodes in `/dev` with appropriate permissions
- Provides device information to user-space applications

## Integration

In Kyronix, the devfs provides static device nodes at boot. udev extends this with dynamic device management for hot-plug events.

> **Note:** The udev implementation is part of the Managarm-derived server infrastructure. A stub (`scripts/libudev_stub.c`) is provided for Xorg compatibility.
