# mbus

This document describes the message bus (mbus) in Kyronix.

## Overview

mbus is a message bus server that handles device discovery and registration. It provides a centralized mechanism for drivers and servers to discover and communicate with each other.

## Architecture

- Drivers register devices with mbus
- Servers query mbus for available devices
- mbus maintains a registry of all system devices

## Integration

mbus communicates with the kernel via standard IPC mechanisms. It receives device notifications from the kernel's device discovery process (PCI enumeration, etc.).

> **Note:** The mbus implementation is part of the Managarm-derived server infrastructure.
