# Building

This document describes the build process for the Kyronix's kernel (codename `k9`). It is the root of the [Building](index.md) section, linking to containerized and native build methods.

## Prerequisites

The kernel targets the x86_64 architecture and requires the following toolchain:

- C11 compiler (`gcc` or `x86_64-elf-gcc`)
- GNU `ld` (or `x86_64-elf-ld`)
- `make`
- `xorriso` (for ISO generation)
- `mkfs.ext2` (for disk image creation)
- `cpio` (for initrd packaging)

The build system uses Kconfig for kernel configuration. The generated header `kernel/config.h` is auto-created from `kernel/Kconfig` via `scripts/kconfig/conf`.

## Build Targets

| Target | Description |
|---|---|
| `make all` | Builds kernel ELF, initrd, and disk image (default) |
| `make iso` | Builds persistent bootable ISO |
| `make live-iso` | Builds live ISO (no persistent disk) |
| `make test-iso` | Builds test ISO with testrunner initrd |
| `make test-disk` | Creates a 16 MiB ext2 test disk image |
| `make test-initrd` | Builds test initrd with userspace test suite |
| `make kallsyms` | Regenerates the kernel symbol table from `kernel.elf` |
| `make nconfig` | Opens interactive ncurses-based kernel configuration |

## Build Output

| Artifact | Path |
|---|---|
| Kernel ELF | `dist/kernel.elf` |
| Persistent ISO | `dist/kkyronix-<VERSION>-INDEV-amd64.iso` |
| Live ISO | `dist/kkyronix-<VERSION>-INDEV-amd64-live.iso` |
| Test ISO | `dist/kkyronix-<VERSION>-INDEV-amd64-test.iso` |
| Initrd | `dist/initrd.cpio` |
| Disk image | `dist/disk.img` |

## Compiler Flags

The kernel is compiled with the following critical flags:

- `-std=c11 -O2 -ffreestanding -fno-stack-protector`
- `-m64 -march=x86-64 -mno-sse -mno-sse2 -mno-mmx -mno-80387`
- `-mno-red-zone -mcmodel=kernel`
- `-fno-pic -fno-pie -fno-omit-frame-pointer`

IMPORTANT: SSE and MMX are disabled in C flags but enabled at runtime via `cpu_enable_sse()` for FPU context switching. The linker script is `linker.ld`.

## Container Builds

The Makefile supports building inside a container (Podman or Docker) by setting the `CRUNTIME` variable. See [with-cbuildrt](with-cbuildrt.md), [with Docker](with-docker.md), or [without containers](without-containers.md) for details.

```bash
make all CRUNTIME=podman
make iso CRUNTIME=docker
```

## QEMU Run Targets

| Target | Description |
|---|---|
| `make run` | Graphical QEMU with KVM, virtio-net, AHCI disk |
| `make run-serial` | Serial-only QEMU (no display) |
| `make run-disk` | Direct disk boot (no ISO) |
| `make run-uefi` | UEFI boot with OVMF |
| `make live-run` | Live session from live ISO |
| `make test-run` | Test runner in QEMU |

NOTE: Run targets do NOT rebuild. Execute `make iso` (or equivalent) first.

## Testing

The test framework uses a custom `testrunner` init that boots the kernel, runs test binaries from the test initrd, and reports results via serial output. Memory leak detection is performed via `kmemleak` during test runs.

```bash
make test-iso && make test-run-log
```

The `test-run-log` target captures serial output to `test.log` and checks for `ALL TESTS PASSED` and zero `KMEMLEAK` reports.

Last reviewed: 2026-07-22
