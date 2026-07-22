# API

This document describes the internal kernel API surface of the Kyronix kernel. It is the child of [Reference](index.md).

## Memory

- `pmm_alloc` - allocate a physical page
- `pmm_alloc_zeroed` - allocate a zeroed physical page
- `pmm_alloc_contiguous` - allocate contiguous physical pages
- `pmm_free` - free a physical page
- `vmm_map` - map a virtual address
- `vmm_unmap` - unmap a virtual address
- `vmm_protect` - set protection flags on a mapping
- `vmm_space_new` - create a new virtual address space
- `vmm_space_free` - free a virtual address space
- `vmm_switch` - switch to a virtual address space
- `vmm_fork_user` - fork user address space
- `kmalloc` - kernel memory allocate
- `kcalloc` - kernel memory calloc
- `krealloc` - kernel memory reallocate
- `kfree` - kernel memory free

## Process

- `proc_alloc` - allocate a process
- `proc_ref` - increment process reference count
- `proc_unref` - decrement process reference count
- `proc_find` - find a process by PID (Process ID)
- `proc_do_exit` - terminate a process
- `sched_switch` - context switch to another process
- `sched_claim_next` - claim the next runnable process

## Filesystem

- `vfs_init` - initialize the Virtual File System (VFS)
- `vfs_lookup` - look up a path in the VFS
- `vfs_mount` - mount a filesystem
- `vfs_node_unref_internal` - unreference an internal VFS node
- `vfs_sync_all` - synchronize all mounted filesystems

## Signals

- `proc_send_signal` - send a signal to a process
- `signal_check` - check for pending signals

## Jail

- `jail_init` - initialize the jail subsystem
- `jail_create` - create a new jail
- `jail_enter` - enter a jail
- `jail_remove` - remove a jail
- `jail_can_see` - check if a process can see another process
- `jail_host_priv` - check if a process has host privileges

## Crypto

- `chacha20_rng_init` - initialize the ChaCha20 random number generator
- `chacha20_rng_bytes` - generate random bytes

Last reviewed: 2026-07-22
