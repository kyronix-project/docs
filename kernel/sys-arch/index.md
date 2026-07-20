# System Architecture

This section describes the high-level architecture of the Kyronix kernel and its surrounding components. Kyronix is a monolithic, hybrid Linux-compatible operating system targeting x86-64.

## Design Philosophy

Kyronix is designed as a Linux-compatible kernel with the following goals:

- **Linux binary compatibility:** Support for 150+ Linux syscalls, enabling execution of unmodified Linux binaries compiled with musl libc.
- **Preemptive multitasking:** Time-sliced preemptive scheduling with per-CPU idle processes.
- **Simplicity:** A straightforward monolithic design without microkernel message passing overhead.
- **Security:** Built-in jail (sandbox) subsystem for process isolation.

## Components

See [System Components](components.md) for a detailed overview of all major subsystems.

## Kernel Subsystems

- [Kernel](kernel/index.md) -- core kernel subsystems (boot, arch, mm, proc, fs, drivers, net, syscall)

## Libraries

- [Libraries](lib/index.md) -- kernel and userspace libraries (frigg, bragi, hel, helix, libasync, mlibc)

## Servers and Drivers

- [Servers](servers/index.md) -- userspace server processes (init, mbus, posix-subsystem, udev)
- [Drivers](drivers/index.md) -- userspace driver framework (libblockfs)
