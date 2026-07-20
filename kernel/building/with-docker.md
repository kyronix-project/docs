# Building with Docker/Podman

Kyronix supports building inside a Docker or Podman container for a fully reproducible environment.

## Prerequisites

1. Install Docker or Podman on your host system.
2. Ensure your user has permission to run containers (e.g., membership in the `docker` group).

## Build Steps

1. Clone the repository and enter the source directory:

```bash
git clone https://github.com/kyronix-project/kyronix.git
cd kyronix
```

2. Build and run using Podman (or replace `podman` with `docker`):

```bash
make all CRUNTIME=podman
```

This will:

1. Build the container image from the `Containerfile` (Alpine 3.20 base).
2. Mount the source directory into the container.
3. Run the full build inside the container.

3. The build output will be placed in `dist/kernel.elf` and `build/bin/`.

## Container Image

The `Containerfile` defines the build environment:

- Base image: Alpine 3.20
- Toolchain: `musl-gcc`, `nasm`, `xorriso`
- Testing: `qemu-system-x86_64`

> **Important:** If the `Containerfile` changes, the container image will be automatically rebuilt on the next `make` invocation.

## Running Tests in the Container

```bash
make test-iso CRUNTIME=podman
make test-run-log CRUNTIME=podman
```
