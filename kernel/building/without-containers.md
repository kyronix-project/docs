# Building Without Containers

This document describes how to build the Kyronix kernel natively on the host system without containers.

## Required Packages

Install the following packages for your distribution:

- **Alpine**: `gcc g++ make binutils musl-dev linux-headers nasm bison flex ncurses-dev cpio e2fsprogs xorriso`
- **Debian/Ubuntu**: `gcc g++ make binutils nasm bison flex libncurses-dev cpio e2fsprogs xorriso`
- **Arch**: `gcc make binutils nasm bison flex ncurses cpio e2fsprogs xorriso`

## Build Steps

1. Ensure `gcc` (or `x86_64-elf-gcc`) and `ld` (or `x86_64-elf-ld`) are in `PATH`.
2. Run the desired target without `CRUNTIME`:

```bash
make all
make iso
```

3. The Kconfig toolchain (`scripts/kconfig/conf`) is built automatically from source on first run.

## Cross-Compilation

If `x86_64-elf-gcc` and `x86_64-elf-ld` are found in `PATH`, the build system uses them automatically. Otherwise, it falls back to the system `gcc` and `ld`.

## Kernel Configuration

Edit kernel options interactively:

```bash
make nconfig
```

This opens an ncurses interface. Configuration is stored in `.config` and generates `kernel/config.h` via Kconfig autoheader.

## Formatting

Format all kernel C source files:

```bash
make fmt
```

Check formatting without modifying files:

```bash
make fmt-check
```

Formatting uses `clang-format` with the project's `.clang-format` style file.

Last reviewed: 2026-07-22
