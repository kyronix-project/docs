# Getting Started

This section provides instructions for building and running the Kyronix kernel from source. Kyronix is a hybrid Linux-compatible operating system targeting x86-64, featuring a preemptive scheduler, 150+ Linux-compatible syscalls, and musl libc.

## Prerequisites

Before building, ensure you have the following tools installed on your host system:

- `gcc` or `x86_64-elf-gcc` cross-compiler
- `nasm` (assembler)
- `musl-gcc` (for user-space binaries)
- `xorriso` (for ISO creation)
- `qemu-system-x86_64` (for testing, optional)
- `make`

Alternatively, you can use the containerized build environment, which requires only `podman` or `docker`.

## Quick Start

1. Clone the repository:

```bash
git clone https://github.com/kyronix-project/kyronix.git
cd kyronix
```

2. Build the kernel and user-space:

```bash
make
```

3. Run in QEMU:

```bash
make run
```

## Build Options

Kyronix supports multiple build and run configurations. See the following sections for details:

- [Building with cbuildrt](../building/with-cbuildrt.md)
- [Building with Docker/Podman](../building/with-docker.md)
- [Building without containers](../building/without-containers.md)

## Documentation Structure

The documentation is organized into the following sections:

- [System Architecture](../sys-arch/index.md) -- high-level design and component overview
- [Implementation Notes](../impl-notes/index.md) -- detailed kernel and driver internals
- [Reference](../reference/index.md) -- syscall and API reference
- [Contributing](../contributing/index.md) -- guidelines for contributing to the project
