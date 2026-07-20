# Building without Containers

This section describes how to build Kyronix natively on the host system without using containers.

## Prerequisites

Install the following packages on your host:

- `gcc` (with x86-64 target support) or `x86_64-elf-gcc`
- `musl-gcc` (musl cross-compiler for user-space)
- `nasm` (Netwide Assembler)
- `xorriso` (for ISO creation)
- `make`

On Alpine Linux:

```bash
apk add gcc musl-dev nasm xorriso make
```

On Debian/Ubuntu:

```bash
apt install gcc nasm xorriso make musl-tools
```

## Build Steps

1. Clone the repository:

```bash
git clone https://github.com/kyronix-project/kyronix.git
cd kyronix
```

2. Build the kernel and user-space:

```bash
make
```

The build system will:

1. Generate `config.h` from the Kconfig configuration.
2. Compile all kernel C and assembly sources.
3. Run `kallsyms` post-processing for kernel symbol resolution.
4. Link the kernel ELF at virtual base `0xffffffff80000000`.
5. Build all user-space binaries with `musl-gcc -static -no-pie`.

3. Run in QEMU:

```bash
make run
```

## Compiler Flags

The kernel is compiled with the following flags:

```
-std=c11 -O2 -ffreestanding -fno-stack-protector -fno-pic -fno-pie
-m64 -march=x86-64 -mno-80387 -mno-mmx -mno-sse -mno-sse2
-mno-red-zone -mcmodel=kernel -fno-omit-frame-pointer
```

> **Warning:** Building without containers may produce different results than the containerized build due to toolchain version differences. Use the containerized build for release artifacts.
