# Jail

This document describes the jail (sandbox) subsystem in the Kyronix kernel.

## Overview

The jail subsystem provides process isolation similar to FreeBSD jails or Linux namespaces. It allows creating sandboxed environments with configurable isolation levels.

## Jail Structure

```c
typedef struct jail {
    int state;            // JAIL_UNUSED, JAIL_ACTIVE, JAIL_DYING
    uint32_t id;          // unique jail ID
    uint32_t parent_id;   // parent jail (0 = host)
    uint32_t flags;       // isolation flags
    char name[32];        // jail name
    char root[256];       // filesystem root path
    uint32_t nprocs;      // current process count
    uint32_t max_procs;   // process limit (0 = unlimited)
    uint32_t creator_uid; // UID of the creator
} jail_t;
```

## Isolation Flags

| Flag | Value | Description |
|------|-------|-------------|
| `JAILF_FS` | 0x01 | Filesystem root isolation |
| `JAILF_PID` | 0x02 | PID namespace isolation |
| `JAILF_IPC` | 0x04 | IPC namespace isolation |
| `JAILF_PRIV` | 0x08 | Restrict privileged operations |
| `JAILF_EXEMPT` | 0x80 | Exempt from auto-isolation |

## Host Jail

Jail ID 0 (`JAIL_HOST`) represents the unconfined host environment. All processes start in the host jail unless explicitly placed in a child jail.

## Syscalls

| Syscall | Description |
|---------|-------------|
| `jail_create(cfg)` | Create a new jail |
| `jail_attach(jid)` | Move current process into a jail |
| `jail_get(jid, info)` | Get jail information |
| `jail_list()` | List all jails |
| `jail_remove(jid)` | Remove a jail |
| `jail_self()` | Get current jail ID |
| `jail_set_auto(enabled)` | Enable/disable auto-isolation |

## Auto-Isolation

When `jail_set_auto(1)` is enabled, every `execve()` automatically creates a new jail for the process. This provides automatic sandboxing without explicit jail creation.

## Path Confinement

Jails with `JAILF_FS` confine filesystem access to the jail's root path:

- `jail_canon_clamp()` -- clamps `..` at the jail root
- `jail_root_current()` -- returns the current jail's root
- `jail_strip_root()` -- strips the jail root prefix for `getcwd`

## Process Lifecycle

1. `jail_create()` allocates a jail slot and sets the root path
2. `jail_enter()` transitions a process into the jail (increments `nprocs`)
3. `jail_ref()` / `jail_unref()` manage the process count
4. `jail_remove()` marks the jail as `JAIL_DYING` if processes remain, or frees it immediately
5. When the last process exits, a dying jail is freed automatically

## Visibility

- `jail_can_see(observer, target)` -- in a PID-isolated jail, processes can only see other processes in the same jail
- `jail_host_priv(p)` -- checks if a process has root privileges and is not restricted by `JAILF_PRIV`

## Limits

| Resource | Limit |
|----------|-------|
| Max jails | 32 |
| Max jail name | 32 characters |
| Max jail root path | 256 characters |
