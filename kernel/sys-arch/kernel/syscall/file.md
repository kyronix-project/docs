# File Operations

This document describes the file operation system calls (syscalls) in the Kyronix kernel. It is the child of [Syscalls](index.md).

## Syscall Table

- 0 `read` - Read data from a file descriptor (fd)
- 1 `write` - Write data to a file descriptor
- 2 `open` - Open a file by path
- 3 `close` - Close a file descriptor
- 4 `stat` - Get file status by path
- 5 `fstat` - Get file status by fd
- 6 `lstat` - Get file status by path (no symlink follow)
- 7 `poll` - Wait for events on fds
- 8 `lseek` - Set file offset for a fd

- 16 `ioctl` - Control device parameters
- 17 `pread64` - Read from fd at offset
- 18 `pwrite64` - Write to fd at offset
- 19 `readv` - Read multiple buffers from fd
- 20 `writev` - Write multiple buffers to fd
- 21 `access` - Check file accessibility by path
- 22 `pipe` - Create a pipe
- 23 `select` - Monitor multiple fds for events

- 32 `dup` - Duplicate a fd
- 33 `dup2` - Duplicate a fd to a specific number
- 40 `sendfile` - Transfer data between fds

- 72 `fcntl` - File descriptor control operations
- 76 `truncate` - Truncate a file to a specified length
- 77 `ftruncate` - Truncate a fd to a specified length
- 78 `getdents64` - Read directory entries

- 79 `getcwd` - Get current working directory
- 80 `chdir` - Change current working directory
- 81 `fchdir` - Change current working directory by fd
- 82 `rename` - Rename a file or directory
- 83 `mkdir` - Create a directory
- 84 `rmdir` - Remove a directory
- 85 `creat` - Create a file
- 86 `link` - Create a hard link
- 87 `unlink` - Remove a file
- 88 `symlink` - Create a symbolic link
- 89 `readlink` - Read a symbolic link

- 90 `chmod` - Change file permissions by path
- 91 `fchmod` - Change file permissions by fd
- 92 `chown` - Change file owner by path
- 93 `fchown` - Change file owner by fd
- 94 `lchown` - Change file owner (no symlink follow)
- 95 `umask` - Set file creation mask

- 137 `statfs` - Get filesystem statistics by path
- 138 `fstatfs` - Get filesystem statistics by fd

- 257 `openat` - Open file relative to directory fd
- 258 `mkdirat` - Create directory relative to directory fd
- 260 `mknodat` - Create device node relative to directory fd
- 261 `fchownat` - Change file owner relative to directory fd
- 262 `newfstatat` - Get file status relative to directory fd
- 263 `unlinkat` - Remove file relative to directory fd
- 264 `renameat` - Rename file relative to directory fds
- 265 `linkat` - Create hard link relative to directory fds
- 266 `symlinkat` - Create symbolic link relative to directory fd
- 267 `readlinkat` - Read symbolic link relative to directory fd
- 268 `fchmodat` - Change file permissions relative to directory fd
- 269 `faccessat` - Check file accessibility relative to directory fd

- 292 `dup3` - Duplicate a fd with flags
- 293 `pipe2` - Create a pipe with flags
- 295 `preadv` - Read from fd at offset with scatter-gather
- 296 `pwritev` - Write to fd at offset with scatter-gather
- 326 `copy_file_range` - Copy data between two fds
- 332 `statx` - Get extended file status
- 334 `close_range` - Close a range of fds

## Path Resolution

All path-based syscalls route through `path_abs()` to resolve user-provided paths to absolute paths. This function applies jail root confinement via `jail_root_current()` and prepends the current working directory (`cwd`) for relative paths.

The `at`-relative syscalls (`openat`, `mkdirat`, `unlinkat`, etc.) use `at_resolve()` for directory fd resolution. This function validates the directory fd and combines it with the relative path.

## Internal Helpers

The kernel provides user pointer validation helpers:

- `uptr_ok(ptr, size)` - Validates a user pointer for read access with automatic page faulting
- `uptr_ok_w(ptr, size)` - Validates a user pointer for write access with automatic page faulting

These helpers ensure user pointers are in valid user address space before dereferencing.

Last reviewed: 2026-07-22
