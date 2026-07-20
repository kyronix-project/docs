# Building with cbuildrt

`cbuildrt` is a container runtime designed for fast, reproducible builds. It provides minimal overhead compared to standard Docker/Podman builds.

## Prerequisites

1. Install `cbuildrt` on your host system.
2. Ensure the Kyronix rootfs is available at the expected path.

## Build Steps

1. Clone the repository and enter the source directory:

```bash
git clone https://github.com/kyronix-project/kyronix.git
cd kyronix
```

2. Run the build using `cbuildrt`:

```bash
make all CRUNTIME=cbuildrt
```

3. The build output will be placed in `dist/kernel.elf` and `build/bin/`.

## Configuration

The `cbuildrt` runtime reads its configuration from the `Containerfile` in the repository root. The container image is based on Alpine 3.20 and includes:

- `musl-gcc` (cross-compiler toolchain)
- `nasm` (assembler)
- `xorriso` (ISO creation)
- `qemu-system-x86_64` (testing)

> **Note:** The `cbuildrt` runtime must be installed and configured separately from Docker/Podman. Consult the `cbuildrt` documentation for installation instructions.
