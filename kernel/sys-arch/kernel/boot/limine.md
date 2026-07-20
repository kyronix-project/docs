# Boot

This section describes how the Kyronix kernel boots using the Limine boot protocol.

## Limine Boot Protocol

Kyronix uses the **Limine boot protocol v3** as its bootloader interface. Limine is a simple, modern bootloader that supports both BIOS and UEFI firmware.

### Boot Configuration

The boot configuration is specified in `limine.conf`:

```
TIMEOUT=3
:Kyronix
    PROTOCOL=limine
    KERNEL_PATH=boot():/boot/kernel.elf
    # MODULE_PATH=boot():/boot/initrd.cpio
    # MODULE_CMDLINE=initrd
```

Variants:

| Config File | Purpose | Initrd |
|-------------|---------|--------|
| `limine.conf` | Default ISO boot | No |
| `limine-disk.conf` | Disk boot | No |
| `limine-live.conf` | Live ISO | Yes (`initrd.cpio`) |
| `limine-test.conf` | Test ISO | Yes (`initrd.cpio`) |

### Limine Requests

The kernel declares the following Limine requests in `kernel.c`:

| Request | Purpose |
|---------|---------|
| `LIMINE_FRAMEBUFFER_REQUEST` | Framebuffer address, width, height, bpp |
| `LIMINE_MEMMAP_REQUEST` | Physical memory map |
| `LIMINE_HHDM_REQUEST` | Higher Half Direct Map offset |
| `LIMINE_MODULE_REQUEST` | Boot modules (initrd) |
| `LIMINE_RSDP_REQUEST` | ACPI RSDP pointer |
| `LIMINE_SMP_REQUEST` | SMP CPU information |

### Entry Point

The kernel entry point is `kmain()`, specified as `ENTRY(kmain)` in the linker script. Limine calls `kmain()` directly after setting up the environment. There is no separate `_start` stub.

### Linker Script Layout

```
PHDRS: text(r-x), rodata(r--), data(rw-)

SECTIONS at 0xffffffff80000000:
  .text -> .rodata -> .limine_requests -> .data -> .bss
```

The kernel is a higher-half kernel mapped at virtual address `0xffffffff80000000`. Physical memory is accessible via the Higher Half Direct Map (HHDM) offset provided by Limine.

### Boot Sequence

See [Initialization](../../impl-notes/kernel/initialization.md) for the detailed step-by-step boot sequence.

### See Also

- [Limine Documentation](https://limine-bootloader.org/)
- [Initialization](../../impl-notes/kernel/initialization.md)
