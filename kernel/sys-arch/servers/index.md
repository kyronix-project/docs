# Servers

This section describes the user-space server processes in the Kyronix system.

## Overview

Kyronix uses a server-based architecture for certain system services. These servers run in user-space and communicate via IPC mechanisms.

## Components

- [Init](init.md) -- PID 1 init process
- [mbus](mbus.md) -- message bus for device discovery
- [posix-subsystem](posix-subsystem.md) -- POSIX compatibility layer
- [udev](udev.md) -- device manager

## Architecture

```
+-------------------+
|    Applications   |
+-------------------+
| posix-subsystem   |  (POSIX syscall emulation)
| udev              |  (device management)
| mbus              |  (device discovery)
+-------------------+
|    Kyronix Kernel |
+-------------------+
```

## Communication

Servers communicate with each other and with the kernel via:

- Unix domain sockets
- The message bus (mbus)
- Standard POSIX IPC (pipes, signals, shared memory)
