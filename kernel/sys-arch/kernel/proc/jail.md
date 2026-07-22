# Jail

This document describes the jail (sandbox) subsystem in the Kyronix kernel. It is the child of [Process Management](sys-arch/kernel/proc/index.md).

## Source

`kernel/proc/jail.c`, `kernel/proc/jail.h`

## Overview

Jails provide process isolation through hierarchical containment with four independent isolation axes: filesystem, PID namespace, IPC, and privilege restriction. Up to 32 jails can exist simultaneously.

## Jail Flags

| Flag | Value | Effect |
|---|---|---|
| `JAILF_FS` | 0x01 | Filesystem isolation (chroot-like root) |
| `JAILF_PID` | 0x02 | PID namespace isolation |
| `JAILF_IPC` | 0x04 | IPC isolation |
| `JAILF_PRIV` | 0x08 | Privilege restriction (root inside jail is restricted) |

## Jail States

| State | Value | Description |
|---|---|---|
| `JAIL_UNUSED` | 0 | Slot is free |
| `JAIL_ACTIVE` | 1 | Jail is operational |
| `JAIL_DYING` | 2 | Pending destruction (waiting for processes to exit) |

## Data Structures

```c
typedef struct {
    int      state;
    uint32_t id;
    uint32_t parent_id;
    uint32_t flags;
    char     name[32];
    char     root[256];
    int      nprocs;
    int      max_procs;
    uint32_t creator_uid;
} jail_t;
```

## Syscalls

| Syscall | Number | Description |
|---|---|---|
| `jail_create` | 500 | Create a new jail |
| `jail_attach` | 501 | Move process into a jail |
| `jail_get` | 502 | Get jail info by ID |
| `jail_list` | 503 | List all visible jails |
| `jail_remove` | 504 | Remove/destroy a jail |
| `jail_self` | 505 | Get current process's jail ID |
| `jail_set_auto` | 506 | Toggle auto-isolation mode |

## Key Functions

| Function | Description |
|---|---|
| `jail_init()` | Zero all jail slots |
| `jail_create(parent_id, cfg, uid)` | Create jail with parent relationship |
| `jail_enter(p, jid)` | Move process into jail |
| `jail_remove(jid, requester)` | Remove jail (permission checked) |
| `jail_can_see(observer, target)` | PID namespace visibility check |
| `jail_host_priv(p)` | Check host-level privilege |
| `jail_can_fork(jid)` | Check fork permission and capacity |
| `jail_root_current()` | Get current process's jail root |
| `jail_canon_clamp(path, sz, root)` | Clamp path within jail root |

## Path Canonicalization

The jail system canonicalizes paths to prevent directory traversal escapes:

1. `path_canon()` resolves `.` and `..`, collapses multiple slashes
2. `jail_canon_clamp()` ensures the result stays within the jail's root
3. `jail_strip_root()` removes the jail root prefix for `getcwd()` display

## Hierarchy

Jails form a tree rooted at `JAIL_HOST` (0). A process can only enter a jail that is a descendant of its current jail. This prevents privilege escalation by entering an ancestor jail.

## Auto-Isolation

When `g_jail_auto_isolate` is enabled (via `jail_set_auto`), every `execve()` call creates a fresh jail for the process. The init process and its direct descendants are exempt (marked with `JAILF_EXEMPT`).

Last reviewed: 2026-07-22
