# Kernel Initialization

This document describes the detailed kernel initialization sequence in the Kyronix kernel. It is the child of [Kernel Implementation Notes](index.md).

The `kmain()` function executes a nine-phase initialization sequence that brings up all kernel subsystems from early hardware detection through to the first userspace process execution.

## Initialization Phases

1. **Phase 1 — Early console and core tables:** `serial_init`, `printf` setup, Global Descriptor Table (GDT) initialization (`gdt_init`), Interrupt Descriptor Table (IDT) initialization (`idt_init`), and keyboard initialization (`kbd_init`).

2. **Phase 2 — Boot protocol validation:** Validate Limine boot protocol responses, compute `kernel_end_phys`, and configure the bootstrap processor (BSP) Model-Specific Registers (MSRs) via `g_cpu_local[0]`.

3. **Phase 3 — Core memory and processor setup:** Physical Memory Manager initialization (`pmm_init`), framebuffer initialization (`fb_init`), Virtual Memory Manager initialization (`vmm_init`), Local APIC (Advanced Programmable Interrupt Controller) initialization (`lapic_init`), Symmetric Multi-Processing initialization (`smp_init`), and Supervisor Mode Execution Prevention (SMEP) enable.

4. **Phase 4 — Kernel services:** Kernel heap initialization (`heap_init`), system call initialization (`syscall_init`), process subsystem initialization (`proc_init`), jail initialization (`jail_init`), Virtual File System initialization (`vfs_init`), and root mount verification.

5. **Phase 5 — Hardware enumeration:** PCI enumeration (`pci_enumerate`), Advanced Configuration and Power Interface (ACPI) initialization (`acpi_init`), block device initialization (`block_init`), AHCI (Advanced Host Controller Interface) initialization (`ahci_init`), VirtIO network initialization (`virtnet_init`), and network stack initialization (`net_init`).

6. **Phase 6 — User interface devices:** User Interface (UI) initialization (`uio_init`), framebuffer device initialization (`fbdev_init`), input subsystem initialization (`input_init`), virtual terminal initialization (`vt_init`), Programmable Interval Timer (PIT) initialization (`pit_init`), and LAPIC timer calibration (`lapic_calibrate_timer`).

7. **Phase 7 — Application processor boot and entropy:** BSP idle process creation, Application Processor (AP) boot (`smp_boot_aps`), and Cryptographically Secure Pseudo-Random Number Generator (CSPRNG) initialization with RDRAND instruction and Timestamp Counter (TSC) fallback.

8. **Phase 8 — Interrupts and self-tests:** `sti` (Set Interrupt Flag), PS/2 mouse initialization (`ps2mouse_init`), and self-tests for the PMM, VMM, and heap subsystems.

9. **Phase 9 — Filesystem and init:** ext2 filesystem initialization (`ext2_init`), root mount, initial ramdisk (initrd) load, Filesystem Table (fstab) mount, and execution of `/init`.

## Self-Tests

The following self-tests execute during Phase 8 to verify core subsystem integrity.

### PMM Self-Test

1. Allocate four physical pages.
2. Map each page to a virtual address via `phys_to_virt`.
3. Write the constant `0xDEADBEEFCAFEBABE` to each page.
4. Verify that each page contains a unique allocation (distinct physical addresses).

### VMM Self-Test

1. Map a single page at virtual address `0xffff900000001000`.
2. Write the constant `0xC0FFEE00DEADC0DE` to the mapped page.
3. Read back and verify the written value.
4. Unmap the page.

### Heap Self-Test

1. Allocate buffers of 64, 128, and 256 bytes.
2. Fill the 64-byte buffer with `0xAA`, the 128-byte buffer with `0xBB`, and the 256-byte buffer with `0xCC`.
3. Verify each buffer contains the expected fill pattern.
4. Test `krealloc` by reallocating each buffer and verifying content preservation.

Last reviewed: 2026-07-22
