# CPU Primitives

This document describes the low-level CPU primitives used throughout the Kyronix kernel.

## Port I/O

The kernel provides inline functions for x86 port I/O operations:

| Function | Description |
|----------|-------------|
| `inb(port)` | Read a byte from a port |
| `outb(port, val)` | Write a byte to a port |
| `inw(port)` | Read a word (16-bit) from a port |
| `outw(port, val)` | Write a word to a port |
| `inl(port)` | Read a dword (32-bit) from a port |
| `outl(port, val)` | Write a dword to a port |
| `io_wait()` | Short delay via port 0x80 |

## Interrupt Control

| Function | Description |
|----------|-------------|
| `cli()` | Clear interrupt flag (disable interrupts) |
| `sti()` | Set interrupt flag (enable interrupts) |
| `hlt()` | Halt until next interrupt |
| `cpu_relax()` | PAUSE instruction (spin-loop hint) |
| `cpu_halt()` | cli + infinite hlt loop |

## MSR Access

| Function | Description |
|----------|-------------|
| `rdmsr(msr)` | Read a Model-Specific Register |
| `wrmsr(msr, val)` | Write a Model-Specific Register |

## Control Registers

| Function | Description |
|----------|-------------|
| `read_cr0()` / `write_cr0(val)` | Read/write CR0 |
| `read_cr2()` | Read CR2 (faulting address) |
| `read_cr3()` | Read CR3 (page table base) |
| `read_cr4()` / `write_cr4(val)` | Read/write CR4 |

## SSE/FPU Initialization

`cpu_enable_sse()` performs the following steps:

1. Clear CR0.EM (bit 2) -- disable x87 emulation
2. Set CR0.MP (bit 1) -- monitor coprocessor
3. Set CR4.OSFXSR (bit 9) -- enable FXSAVE/FXRSTOR
4. Set CR4.OSXMMEXCPT (bit 10) -- enable unmasked SSE exceptions
5. Execute `fninit` + load MXCSR with 0x1F80 (mask all exceptions, round to nearest)

## IRQ Save/Restore

| Function | Description |
|----------|-------------|
| `irq_save()` | Push RFLAGS, disable interrupts, return old flags |
| `irq_restore(flags)` | Restore RFLAGS (re-enables interrupts if they were enabled) |

These are used for interrupt-safe critical sections throughout the kernel.

## Type Definitions

The kernel defines the following architecture-specific types:

- `gdt_entry_t` -- 8-byte GDT entry
- `gdtr_t` -- GDT register (limit + base)
- `idt_entry_t` -- 16-byte IDT entry
- `idtr_t` -- IDT register (limit + base)
- `cpu_state_t` -- Full register frame saved on interrupt/syscall entry (r15..rax, int_no, error_code, rip, cs, rflags, rsp, ss)
