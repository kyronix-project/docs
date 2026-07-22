# Boot

This document describes the Kyronix kernel boot process. It is the child of [Kernel](sys-arch/kernel/index.md) and parent of the Limine protocol reference.

## Boot Sequence

The Kyronix kernel boots via the Limine v3 boot protocol. The bootloader loads the kernel ELF and provides the following information through Limine requests:

1. **Memory map** (`LIMINE_MEMMAP_REQUEST`) -- Physical memory regions (usable, reserved, ACPI, framebuffer)
2. **HHDM offset** (`LIMINE_HHDM_REQUEST`) -- Higher Half Direct Map base address
3. **Framebuffer** (`LIMINE_FRAMEBUFFER_REQUEST`) -- Linear framebuffer for early display
4. **Kernel address** (`LIMINE_KERNEL_ADDRESS_REQUEST`) -- Physical and virtual base addresses
5. **RSDP** (`LIMINE_RSDP_REQUEST`) -- ACPI Root System Description Pointer
6. **Modules** (`LIMINE_MODULE_REQUEST`) -- Bootloader modules (initrd)

All requests use `LIMINE_BASE_REVISION(3)`.

## Boot Phases

### Phase 1: Early Hardware (BSP)

1. `serial_init(COM1)` -- Initialize serial port for debug output
2. `gdt_init()` -- Set up Global Descriptor Table and Task State Segment
3. `idt_init()` -- Set up Interrupt Descriptor Table, remap PIC to vectors 32-47
4. `kbd_init()` -- Initialize keyboard driver

### Phase 2: Memory Setup

1. `pmm_init()` -- Initialize physical memory manager from Limine memory map
2. `vmm_init()` -- Enable NX bit, initialize kernel page tables
3. `heap_init()` -- Initialize kernel heap allocator (64 KiB initial)

### Phase 3: Per-CPU and SMP

1. Write `MSR_GS_BASE` and `MSR_KERNEL_GS_BASE` for BSP per-CPU data
2. `smp_init()` -- Enumerate CPUs from Limine SMP response
3. SMEP detection and enable via CPUID leaf 7
4. `syscall_init()` -- Configure SYSCALL/SYSRET MSRs, enable SSE

### Phase 4: Device Drivers

1. `pci_enumerate()` -- Scan PCI bus
2. `acpi_init()` -- Parse ACPI tables from RSDP
3. `ahci_init()` -- Initialize SATA/AHCI controllers
4. `virtnet_init()` -- Initialize VirtIO network device
5. `net_init()` -- Initialize lwIP network stack

### Phase 5: Timer and Scheduling

1. `pit_init()` -- Program PIT channel 0 at ~250 Hz
2. `lapic_calibrate_timer()` -- Calibrate LAPIC timer against PIT
3. Create BSP idle process
4. `smp_boot_aps()` -- Wake Application Processors

### Phase 6: Filesystem and Init

1. `ext2_init()` -- Register ext2 filesystem driver
2. Mount root filesystem from disk or load initrd via CPIO
3. `process_exec("/init")` -- Load and execute init process

## AP Boot Sequence

Application Processors are woken via the Limine SMP `goto_address` trampoline (`ap_trampoline` in `kernel/proc/ap_trampoline.S`). Each AP:

1. Loads its `cpu_local_t` from `limine_smp_info.extra_argument`
2. Loads the idle process kernel stack
3. Calls `ap_init_cpu()` which initializes GDT, IDT, MSRs, SSE, SYSCALL, LAPIC
4. Spins on `g_kernel_ready` until BSP signals completion
5. Starts 250 Hz LAPIC periodic timer
6. Enters `ap_sched_loop()` (idle/scheduling loop)

## Status Output

During boot, each initialization step prints a status line:

```
* Initialising PMM ...                                              [ ok ]
* Initialising VMM ...                                              [ ok ]
```

The `[ ok ]` / `[ !! ]` indicator is right-aligned at column 72 using ANSI escape codes.

Last reviewed: 2026-07-22
