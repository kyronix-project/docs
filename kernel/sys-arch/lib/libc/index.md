# C Library (libc)

This section describes the C standard library implementation used in Kyronix.

## mlibc

Kyronix uses **musl libc** as its C standard library. Musl is a lightweight, fast, and standards-conforming libc implementation.

### Features

- Full C11 and POSIX.1 support
- Static linking (no dynamic linker needed for most programs)
- Small binary size
- Fast startup time

### Integration

User-space programs are compiled with:

```bash
musl-gcc -static -no-pie -o program program.c
```

### Dynamic Linker

For programs that need dynamic linking, musl provides `ld-musl-x86_64.so.1` in the root filesystem.

## Syscall Wrappers

Musl libc provides thin wrappers around the kernel syscalls. The Kyronix kernel implements the Linux x86-64 syscall ABI, so musl's syscall wrappers work directly:

```c
// musl syscall wrapper example
long syscall(long number, ...) {
    // Assembly: mov rax, number; syscall; ret
}
```

## Supported Functionality

The following libc functionality is fully supported:

- String functions (string.h)
- Standard I/O (stdio.h) -- limited to stderr (serial) and file-backed I/O
- Memory allocation (stdlib.h)
- Math functions (math.h)
- POSIX threads (pthread.h) -- via clone(CLONE_THREAD)
- Socket functions (sys/socket.h) -- via inet and unix socket syscalls
- Signal functions (signal.h)
- Time functions (time.h)
