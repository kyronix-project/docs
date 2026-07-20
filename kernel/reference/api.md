# API Reference

This document provides a reference for the Kyronix kernel's internal API.

## Memory Management

### PMM (Physical Memory Manager)

```c
void pmm_init(struct limine_memmap_response *mmap, uint64_t hhdm_offset);
void *pmm_alloc(void);
void *pmm_alloc_zeroed(void);
void *pmm_alloc_contiguous(uint64_t n_pages);
void pmm_free(void *phys);
uint64_t pmm_free_pages(void);
uint64_t pmm_total_pages(void);
```

### VMM (Virtual Memory Manager)

```c
void vmm_init(void);
int vmm_map(vmm_space_t *sp, uint64_t virt, uint64_t phys, uint64_t flags);
void vmm_unmap(vmm_space_t *sp, uint64_t virt);
uint64_t vmm_virt_to_phys(vmm_space_t *sp, uint64_t virt);
bool vmm_user_range_ok(vmm_space_t *sp, uint64_t virt, uint64_t len, bool write);
bool vmm_user_range_fault_in(vmm_space_t *sp, uint64_t virt, uint64_t len, bool write);
int vmm_protect(vmm_space_t *sp, uint64_t virt, uint64_t flags);
vmm_space_t *vmm_space_new(void);
void vmm_space_free(vmm_space_t *sp);
void vmm_switch(vmm_space_t *sp);
int vmm_fork_user(vmm_space_t *dst, vmm_space_t *src);
```

### Heap

```c
void heap_init(void);
void *kmalloc(uint64_t size);
void *kcalloc(uint64_t nmemb, uint64_t size);
void *krealloc(void *ptr, uint64_t new_size);
void kfree(void *ptr);
```

### VMA

```c
int vma_add(vmm_space_t *sp, uint64_t start, uint64_t len, uint32_t prot,
            uint32_t map_flags, bool free_on_unmap);
int vma_remove(vmm_space_t *sp, uint64_t start, uint64_t len);
int vma_protect(vmm_space_t *sp, uint64_t start, uint64_t len, uint32_t prot);
bool vma_page_fault_allowed(vmm_space_t *sp, uint64_t addr, bool write, bool exec);
uint64_t vma_page_flags(vmm_space_t *sp, uint64_t addr);
```

## Process Management

```c
void proc_init(void);
proc_t *proc_alloc(uint32_t ppid);
proc_t *proc_find(uint32_t pid);
proc_t *proc_create_idle(uint32_t cpu_id, void (*entry)(void));
void proc_do_exit(int code);
void sched_switch(proc_t *next);
void sched_yield_blocking(void);
```

## Filesystem

```c
void vfs_init(void);
int fd_open(const char *path, int flags, int mode);
int fd_close(int fd);
int64_t fd_read(int fd, void *buf, uint64_t len);
int64_t fd_write(int fd, const void *buf, uint64_t len);
int fd_pipe(int pipefd[2]);
int fd_socket(int domain, int type, int proto);
```

## Drivers

```c
void pci_enumerate(void);
void acpi_init(uint64_t rsdp_phys);
void serial_init(uint16_t port);
void pit_init(void);
void lapic_init(void);
```

## Logging

```c
void log_info(const char *fmt, ...);
void log_warn(const char *fmt, ...);
void log_error(const char *fmt, ...);
void log_debug(const char *fmt, ...);
```

## Synchronization

```c
void spin_lock(spinlock_t *lock);
void spin_unlock(spinlock_t *lock);
void spin_lock_irqsave(spinlock_t *lock, uint64_t flags);
void spin_unlock_irqrestore(spinlock_t *lock, uint64_t flags);
uint64_t irq_save(void);
void irq_restore(uint64_t flags);
```
