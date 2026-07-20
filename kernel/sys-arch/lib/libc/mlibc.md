# mlibc

This document describes the musl libc integration in Kyronix.

## Overview

Kyronix uses musl libc as its standard C library. Musl is statically linked for most user-space programs, providing a lightweight and standards-conforming runtime.

## Key Characteristics

- **Static linking:** Most programs are linked with `-static`, avoiding dynamic linker overhead
- **Small footprint:** musl produces much smaller binaries than glibc
- **POSIX compliance:** Full POSIX.1-2008 and C11 support
- **Fast startup:** Minimal initialization overhead

## Build Configuration

User-space binaries are built with:

```bash
musl-gcc -static -no-pie -o <binary> <sources>
```

The `-no-pie` flag is required because the kernel's execve loader places ELF binaries at fixed addresses.

## Dynamic Linking

For programs that require dynamic linking (e.g., Xorg, complex applications), the musl dynamic linker `ld-musl-x86_64.so.1` is included in the root filesystem at `/lib/`.

## Limitations

- No NSS (Name Service Switch) -- DNS resolution is limited
- No locale support
- No iconv (character encoding conversion)
- Thread-local storage uses the `arch_prctl(ARCH_SET_FS)` mechanism

> **Note:** Musl was chosen for its simplicity, small size, and compatibility with the Kyronix minimal system design.
