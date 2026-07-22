# Building with Docker or Podman

This document describes how to build the Kyronix kernel inside a Docker or Podman container. This is the recommended method for reproducible builds.

## Overview

The containerized build uses the `Containerfile` at the repository root, based on Alpine Linux 3.20. The container image includes all required build dependencies: `gcc`, `make`, `binutils`, `nasm`, `bison`, `flex`, `cpio`, `e2fsprogs`, `xorriso`, and others.

## Usage

1. Build any target with the `CRUNTIME` variable set:

```bash
make iso CRUNTIME=podman
make all CRUNTIME=docker
```

2. The Makefile automatically builds the container image from `Containerfile` if it does not exist or if the `Containerfile` has been modified since the image was last built.

3. The source tree is mounted at `/src` inside the container. All build artifacts are written to the host filesystem.

## Supported Runtimes

| Runtime | Variable | Default |
|---|---|---|
| Podman | `CRUNTIME=podman` | Yes (default in Makefile) |
| Docker | `CRUNTIME=docker` | No |

## Container Image

The image is tagged `kyronix-build:latest` by default. Override via `CONTAINER_IMAGE` and `CONTAINER_IMAGE_TAG` variables.

```bash
make iso CRUNTIME=docker CONTAINER_IMAGE=custom-build CONTAINER_IMAGE_TAG=v1
```

## Cleanup

```bash
make clean CRUNTIME=podman
```

This removes the `dist/`, `build/`, and `iso_root/` directories on the host, plus rebuilds the container image on the next build.

NOTE: The container image is cached between builds. Only rebuilds when `Containerfile` changes.

Last reviewed: 2026-07-22
