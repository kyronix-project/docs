# Bragi

This document describes the bragi IPC protocol library.

## Overview

Bragi is a serialization protocol used for Inter-Process Communication (IPC) between kernel servers. It defines message formats for communication between the posix-subsystem, mbus, and other servers.

## Usage

Bragi messages are used for:

- Device discovery (mbus)
- POSIX syscall forwarding
- Server-to-server communication

> **Note:** Bragi is a dependency inherited from the Managarm project. It is used by the user-space server components.
