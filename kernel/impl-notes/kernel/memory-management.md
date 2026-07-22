# Memory Management Implementation

This document describes the implementation details of the Kyronix memory management subsystem.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│  pmm.c   — Physical page allocator (kernel-facing API)     │
│            Wraps LLFree; page-level alloc/free              │
├─────────────────────────────────────────────────────────────┤
│  llfree.c — Lock-free hierarchical allocator core          │
│              local → trees → lower                          │
├──────────┬──────────────────┬───────────────────────────────┤
│ local.c  │    trees.c       │        lower.c                │
│ per-CPU  │ tree array mgmt  │  bitfield + child mgmt        │
│ reserves │ atomic CAS ops   │  actual frame allocation      │
├──────────┴──────────────────┴───────────────────────────────┤
│  tree.c   │  bitfield.c  │  child.c                         │
│  4-byte   │  512-bit     │  2-byte                          │
│  entries  │  atomics     │  counters                        │
└───────────┴──────────────┴──────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  vmm.c  — x86_64 4-level page table management             │
│  vma.c  — VMA tracking, fault-in, protect, fork            │
├─────────────────────────────────────────────────────────────┤
│  heap.c — First-fit kernel heap, grows via PMM             │
├─────────────────────────────────────────────────────────────┤
│  shm.c      — SysV shared memory segments                  │
│  kmemleak.c — Runtime leak detection via pointer scanning  │
└─────────────────────────────────────────────────────────────┘
```

## Physical Memory Manager (PMM)

`pmm.c` / `pmm.h`

The PMM provides a kernel-facing page allocation API built on top of the LLFree lock-free allocator.

### Constants

- `PAGE_SIZE` = 4096, `PAGE_SHIFT` = 12
- `phys_to_virt(phys)` = `phys + g_hhdm_offset`
- `virt_to_phys(virt)` = `virt - g_hhdm_offset`

### Initialization

`pmm_init()` receives the Limine memory map, HHDM offset, and kernel end address. It:

1. Finds the largest usable memory region.
2. Computes metadata sizes for LLFree (local, trees, lower buffers).
3. Allocates metadata from a suitable memory map entry.
4. Initializes the LLFree allocator (`LLFREE_INIT_FREE` mode).
5. Fills a zero-page pool (up to 32 pre-zeroed pages).

### Allocation

| Function | Description |
|---|---|
| `pmm_alloc()` | Allocates one 4 KiB page (class 0, order 0, local=current CPU) |
| `pmm_alloc_zeroed()` | Returns a zeroed page — first tries the zero-page pool, then falls back to `pmm_alloc` + `memset` |
| `pmm_alloc_contiguous(n)` | Allocates `n` contiguous pages by rounding up to the next power-of-2 order |

### Deallocation

`pmm_free(void *phys)` returns a page to the allocator.

### Statistics

- `pmm_free_pages()` — current free page count
- `pmm_total_pages()` — total pages in system
- `pmm_usable_pages()` — total usable pages
- `pmm_alloc_total()` / `pmm_free_total()` — lifetime counters

## LLFree Allocator

The physical page allocator uses **LLFree** (`llfree.c`, `llfree.h`), a lock-free hierarchical buddy-like allocator. Credits to Luhsra for the open-source implementation ([llfree-c](https://github.com/luhsra/llfree-c)).

### Hierarchy

```
llfree_t (core)
  ├── lower_t       — bitfields (512-bit atomic bitmaps) + child counters
  ├── local_t       — per-CPU-per-class reservation entries
  ├── trees_t       — array of atomic tree_t entries
  └── policy_fn     — class-matching callback
```

### Frame Layout Constants

| Constant | Value | Meaning |
|---|---|---|
| `LLFREE_FRAME_SIZE` | 4096 | Base frame size (4 KiB) |
| `LLFREE_FRAME_BITS` | 12 | Bits for frame offset |
| `LLFREE_HUGE_ORDER` | 9 | Log2 of huge frame (512 × 4K = 2 MiB) |
| `LLFREE_CHILD_SIZE` | 512 | Frames per child (bitfield) |
| `LLFREE_TREE_CHILDREN` | 8 | Children per tree |
| `LLFREE_TREE_SIZE` | 4096 | Frames per tree (8 × 512) |
| `LLFREE_MAX_ORDER` | 12 | Maximum allocatable order |
| `LLFREE_MAX_CLASSES` | 8 | Maximum memory classes |

### Key Data Structures

**`tree_t`** (4 bytes) — One entry in the tree array, covering 4096 frames:
```c
typedef struct tree {
    bool reserved : 1;          // Is this tree reserved by a CPU?
    uint8_t class : 3;          // Page class (0..7)
    treeF_t free : 28;          // Free frame count
} tree_t;
```

**`child_t`** (2 bytes) — Index entry for one bitfield (512 frames):
```c
typedef struct child {
    uint16_t free : 15;         // Free base-frame count (0..512)
    bool huge : 1;              // Allocated as a single huge page
} child_t;
```

**`bitfield_t`** — 512-bit atomic bitmap (8 × `uint64_t`, cache-line aligned). Tracks individual frame allocation states within a child.

**`reserved_t`** (8 bytes) — Per-CPU-per-class reservation:
```c
typedef struct reserved {
    bool present : 1;           // Is there a reservation?
    treeF_t free : 28;          // Free frames in reserved tree
    uint64_t start_row : 35;    // Bitfield row index
} reserved_t;
```

### Class System

LLFree supports up to 8 memory classes. Each class represents a category of page usage (e.g., movable, immovable, huge). Predefined policies:

- **Simple** (2 classes): small (0) and huge (1)
- **Movable** (3 classes): immovable (0), movable (1), huge (2)

Each class has a configurable number of local reservation slots per CPU.

### Allocation Algorithm

`llfree_get()` performs allocation in stages:

1. **Local reservation**: Try decrementing the current CPU's local reservation for the requested class. If the reserved tree has enough free frames, allocate from it via `lower_get()`.
2. **Local sync**: If the local reservation is empty, attempt `sync_with_global()` — steal frames from the global tree counter back into the local reservation.
3. **Search and reserve**: Two-phase search:
   - **Near-neighbor best-fit**: Search trees near the preferred start frame using alternating-offset linear scan.
   - **Global best-fit**: Search all trees with a priority queue (top-8 candidates).
   - On match: reserve the tree, then allocate from `lower_get()`.
4. **Steal from other CPUs**: If no tree has enough free frames, steal a reservation from another CPU's local entry.
5. **Demote from other classes**: Attempt to demote (reclassify) frames from a different class's reservation.

### Deallocation Algorithm

`llfree_put()`:

1. Return frames to the lower-level bitfield via `lower_put()`.
2. Try to add frames to the current CPU's local reservation.
3. If that fails, return frames to the global tree via `trees_put()`.

### Tree Search

`trees_search()` uses alternating offset linear scan from a start position (offset 0, +1, -1, +2, -2, ...). The best-fit variant `trees_search_best()` maintains a top-8 priority queue.

### Lock-Free Safety

All tree and child operations use atomic CAS (compare-and-swap) via `atom_cmp_exchange()`. The `atom_update()` macro provides a CAS loop: load, apply function, attempt CAS, retry on failure.

## Virtual Memory Manager (VMM)

`vmm.c` / `vmm.h`

### Page Table Walk

`vmm_map()` performs a 4-level x86_64 page table walk (PML4 → PDPT → PD → PT). Intermediate entries are always `PRESENT|WRITE|USER`. Leaf PTEs are set via `vmm_leaf_pte()` with the actual flags.

### Page Table Flags

| Flag | Value | Description |
|---|---|---|
| `VMM_PRESENT` | `1` | Page is present |
| `VMM_WRITE` | `2` | Writable |
| `VMM_USER` | `4` | User-mode accessible |
| `VMM_PCD` | `1 << 4` | Page cache disable |
| `VMM_NX` | `1 << 63` | No-execute |

Composite flag presets: `VMM_KCODE`, `VMM_KDATA`, `VMM_UCODE`, `VMM_UDATA`.

### Address Space Pool

A fixed pool of 256 address spaces. Each new space (`vmm_space_new()`) copies kernel PML4 entries (indices 256–511) from the global kernel space and initializes a VMA array (up to `VMM_VMA_MAX` = 2048 VMAs).

### Context Switch

`vmm_switch()` syncs the kernel half of the target space's PML4 with the global kernel space, then writes CR3.

### Fork

`vmm_fork_user()` performs a deep copy: it copies VMA metadata from source to destination, then walks the source user-half page tables and copies every present page (alloc + memcpy + map). This is a full-copy fork, not COW.

### Translation

`vmm_virt_to_phys()` walks all 4 levels to resolve a virtual address to a physical address. `vmm_user_range_ok()` checks that every page in a range is present and user-accessible. `vmm_user_range_fault_in()` additionally faults in pages via VMA rules.

## Virtual Memory Area (VMA)

`vma.c` / `vma.h`

VMAs track virtual memory regions within an address space, recording protection, ownership, and fault-in behavior.

### Protection Flags

- `PROT_READ` = 0x1, `PROT_WRITE` = 0x2, `PROT_EXEC` = 0x4

### Operations

| Function | Description |
|---|---|
| `vma_add()` | Add a new VMA with given start, length, protection, and map flags |
| `vma_remove()` | Remove VMAs in a range; splits partially overlapping VMAs at boundaries |
| `vma_remove_overlaps()` | Remove all VMAs overlapping a range |
| `vma_protect()` | Split VMAs at range boundaries, re-add with new protection flags |
| `vma_conflicts()` | Check for overlapping VMAs |
| `vma_range_ok()` | Verify entire range is covered by VMAs |
| `vma_page_fault_allowed()` | Check if a page fault would be valid (matches VMA protection) |
| `vma_page_flags()` | Derive PTE flags from the VMA's protection |

### Split Operation

The core operation is `split_for_hole()`: when removing a range from the middle of a VMA, the VMA is split into up to two pieces. Left-only and right-only truncations do not require an extra slot.

## Heap Allocator

`heap.c` / `heap.h`

### Memory Layout

- Start: `0xffff910000000000`
- Max: `0xffff920000000000` (4 GiB addressable range)
- Growth: 16 pages (64 KiB) per expansion

### Block Layout

Each allocation has a 32-byte header:

```c
typedef struct block_hdr {
    uint64_t size;              // Payload size
    uint64_t free;              // Is this block free?
    struct block_hdr *prev;     // Doubly-linked list
    struct block_hdr *next;
} block_hdr_t;
```

Allocations are 16-byte aligned.

### Operations

| Function | Description |
|---|---|
| `kmalloc(size)` | First-fit allocation; grows heap if needed |
| `kcalloc(nmemb, size)` | Allocates and zeroes memory |
| `krealloc(ptr, new_size)` | Reallocates (alloc + copy + free) |
| `kfree(ptr)` | Frees and coalesces adjacent free blocks |
| `heap_alloc_delta()` | Returns total allocated minus total freed |
| `heap_brk()` | Returns current heap break address |
| `heap_walk_used(cb, user)` | Iterates all allocated blocks via callback |
| `heap_stats()` | Logs allocation statistics |

All operations are protected by `g_heap_lock` (spinlock with IRQ save/restore).

### Growth

When no free block fits, `heap_grow()` allocates contiguous physical pages via `pmm_alloc_contiguous()` and maps them into the heap virtual range.

## SysV Shared Memory

`shm.c` / `shm.h`

### Limits

- `SHM_MAX_SEGS` = 64 segments
- `SHM_MAX_PAGES` = 4096 pages per segment (16 MiB)
- `SHM_MAX_ATTACH` = 256 attachments

### Key Data Structures

```c
typedef struct {
    int shmid, key;
    uint64_t size;
    int n_pages;
    void *pages[SHM_MAX_PAGES];
    int ref_count, destroy_pending;
    uint32_t mode, cuid, jail_id;
} shm_seg_t;

typedef struct {
    int shmid;
    uint64_t va;
    uint32_t pid;
    int n_pages;
} shm_attach_t;
```

### Syscalls

| Function | Description |
|---|---|
| `sys_shmget(key, size, flags)` | Create or get a shared memory segment |
| `sys_shmat(shmid, addr, shmflg)` | Attach segment to process address space |
| `sys_shmdt(addr)` | Detach segment from current process |
| `sys_shmctl(shmid, cmd, buf)` | Control operations (IPC_RMID, IPC_STAT) |
| `shm_proc_exit(pid)` | Clean up all attachments on process exit |

### IPC Features

- **Jail isolation**: Processes in different jails (with `JAILF_IPC` flag) cannot access each other's segments.
- **Permission checks**: Standard SysV permission bits with owner/root bypass.
- **Deferred destroy**: Segments marked IPC_RMID are destroyed when the last attachment is detached (refcount → 0).

## Kernel Memory Leak Detector

`kmemleak.c` / `kmemleak.h`

### Limits

- `KMEMLEAK_MAX` = 8192 tracked heap entries
- `KMEMLEAK_PAGE_MAX` = 2048 tracked page entries
- `KMEMLEAK_BT_DEPTH` = 8 backtrace frames

### Tracked Entries

```c
typedef struct {
    void *ptr;
    uint64_t size;
    uint64_t backtrace[KMEMLEAK_BT_DEPTH];
    int bt_depth, reachable, used;
} kmemleak_entry_t;

typedef struct {
    uint64_t phys;
    uint64_t backtrace[KMEMLEAK_BT_DEPTH];
    int bt_depth, reachable, used, perm;
} page_entry_t;
```

### API

| Function | Description |
|---|---|
| `kmemleak_track(ptr, size)` | Track a heap allocation |
| `kmemleak_untrack(ptr)` | Untrack a heap allocation |
| `kmemleak_track_page(phys)` | Track a physical page |
| `kmemleak_untrack_page(phys)` | Untrack a physical page |
| `kmemleak_page_perm(phys)` | Mark page as permanently reachable (excluded from leak reports) |
| `kmemleak_report(buf, bufsz)` | Scan for leaks, write report, return leak count |
| `kmemleak_checkpoint()` | Scan, return count, clear all entries |
| `kmemleak_clear()` | Clear without scanning |

### Scan Algorithm

`do_scan()`:

1. Reset all `reachable` flags.
2. Scan kernel `.data` and `.bss` sections for pointers.
3. Scan process table and kernel stacks.
4. Scan heap allocations via `heap_walk_used()`.
5. Walk all page tables (kernel + per-process user) marking referenced pages.
6. Any tracked entry or page not marked reachable is reported as a leak.

### Backtrace Capture

Backtraces are captured via frame pointer walking (`rbp` chain), with validation that each frame address is within a mapped kernel region.
