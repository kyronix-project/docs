# Building

This section covers the various methods for building the Kyronix kernel and user-space components. Kyronix supports both native builds on the host system and containerized builds for reproducibility.

## Build Targets

The build system produces the following outputs:

| Output | Path | Description |
|--------|------|-------------|
| Kernel ELF | `dist/kernel.elf` | Linked kernel binary |
| User binaries | `build/bin/` | Statically-linked user-space executables |
| Initrd | `initrd.cpio` | CPIO archive of the root filesystem |
| Bootable ISO | `dist/kyronix.iso` | BIOS+UEFI bootable ISO image |

## Build Methods

1. [Building with cbuildrt](with-cbuildrt.md) -- recommended for fast containerized builds
2. [Building with Docker/Podman](with-docker.md) -- standard containerized build
3. [Building without containers](without-containers.md) -- native build on the host

> **Tip:** The containerized build is recommended for reproducibility. Use the native build only if you need to iterate quickly on kernel changes.

## Common Make Targets

| Target | Description |
|--------|-------------|
| `make` | Build kernel and user-space |
| `make iso` | Create bootable ISO |
| `make run` | Build and run in QEMU |
| `make run-serial` | Run with serial console output |
| `make run-uefi` | Run with UEFI firmware |
| `make test-iso` | Build test ISO |
| `make test-run` | Build and run tests in QEMU |
| `make test-run-log` | Run tests and capture output |
| `make fmt` | Format source code |
| `make clean` | Remove build artifacts |
| `make nconfig` | Open kernel configuration menu |
