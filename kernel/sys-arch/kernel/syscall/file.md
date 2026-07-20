# File Operations

This document describes the file-related syscalls in the Kyronix kernel.

## Syscalls

| Syscall | Number | Description |
|---------|--------|-------------|
| `read` | 0 | Read from file descriptor |
| `write` | 1 | Write to file descriptor |
| `open` | 2 | Open a file |
| `close` | 3 | Close a file descriptor |
| `stat` | 4 | Get file status |
| `fstat` | 5 | Get file status by FD |
| `lstat` | 6 | Get file status (no follow symlink) |
| `poll` | 7 | Poll file descriptors |
| `lseek` | 8 | Seek within a file |
| `mmap` | 9 | Map files into memory |
| `access` | 21 | Check file accessibility |
| `pipe` | 22 | Create a pipe |
| `select` | 23 | Synchronous I/O multiplexing |
| `sched_yield` | 24 | Yield the CPU |
| `dup` | 32 | Duplicate file descriptor |
| `dup2` | 33 | Duplicate file descriptor to specific number |
| `nanosleep` | 35 | Sleep for a specified time |
| `getpid` | 39 | Get process ID |
| `socket` | 41 | Create a socket |
| `connect` | 42 | Initiate a connection |
| `accept` | 43 | Accept a connection |
| `sendto` | 44 | Send a message |
| `recvfrom` | 45 | Receive a message |
| `sendmsg` | 46 | Send a message with ancillary data |
| `recvmsg` | 47 | Receive a message with ancillary data |
| `shutdown` | 48 | Shut down a socket |
| `bind` | 49 | Bind a socket |
| `listen` | 50 | Listen for connections |
| `getsockname` | 51 | Get socket name |
| `getpeername` | 52 | Get peer name |
| `setsockopt` | 54 | Set socket option |
| `getsockopt` | 55 | Get socket option |
| `clone` | 56 | Create a process/thread |
| `fork` | 57 | Fork a process |
| `execve` | 59 | Execute a program |
| `exit` | 60 | Exit the current process |
| `wait4` | 61 | Wait for process state change |
| `kill` | 62 | Send a signal to a process |
| `uname` | 63 | Get system information |
| `fcntl` | 72 | File control |
| `getcwd` | 79 | Get current working directory |
| `chdir` | 80 | Change working directory |
| `mkdir` | 83 | Create a directory |
| `rmdir` | 84 | Remove a directory |
| `creat` | 85 | Create a file |
| `unlink` | 87 | Remove a link |
| `readlink` | 89 | Read a symbolic link |
| `rename` | 82 | Rename a file |
| `truncate` | 93 | Truncate a file |
| `ftruncate` | 94 | Truncate a file by FD |
| `getdents64` | 217 | Read directory entries |
| `openat` | 257 | Open relative to directory FD |
| `mkdirat` | 258 | Create directory relative to FD |
| `newfstatat` | 262 | Stat relative to FD |
| `unlinkat` | 263 | Unlink relative to FD |
| `renameat` | 264 | Rename relative to FD |
| `readlinkat` | 265 | Readlink relative to FD |
| `fchmodat` | 268 | Change file mode relative to FD |
| `accessat` | 269 | Access check relative to FD |
| `dup3` | 292 | Duplicate FD to specific number with flags |
| `pipe2` | 293 | Create a pipe with flags |
| `pread64` | 17 | Read from file at offset |
| `pwrite64` | 18 | Write to file at offset |
| `sendfile` | 40 | Transfer between file descriptors |
| `statx` | 332 | Extended file status |
| `close_range` | 334 | Close a range of FDs |

## File Status

The `linux_stat` structure returned by stat syscalls:

```c
struct linux_stat {
    uint64_t st_dev;
    uint64_t st_ino;
    uint64_t st_nlink;
    uint32_t st_mode;
    uint32_t st_uid;
    uint32_t st_gid;
    uint64_t st_rdev;
    int64_t  st_size;
    int64_t  st_blksize;
    int64_t  st_blocks;
    uint64_t st_atime_sec, st_atime_nsec;
    uint64_t st_mtime_sec, st_mtime_nsec;
    uint64_t st_ctime_sec, st_ctime_nsec;
};
```
