# CPU Primitives

This document describes the x86-64 CPU primitives used by the Kyronix kernel. It is the child of [x86-64 Architecture](sys-arch/kernel/arch/x86_64/index.md).

## Source

`kernel/arch/x86_64/cpu.h`

## Compiler Attributes

| Attribute | Definition | Purpose |
|---|---|---|
| `NORETURN` | `__attribute__((noreturn))` | Function never returns |
| `PACKED` | `__attribute__((packed))` | No struct padding |
| `ALIGNED(n)` | `__attribute__((aligned(n)))` | Alignment requirement |
| `INLINE` | `static inline __attribute__((always_inline))` | Force inlining |
| `UNUSED` | `__attribute__((unused))` | Suppress unused warnings |

## I/O Port Functions

| Function | Description |
|---|---|
| `outb(port, val)` | Write byte to I/O port |
| `outw(port, val)` | Write 16-bit word to I/O port |
| `outl(port, val)` | Write 32-bit dword to I/O port |
| `inb(port)` | Read byte from I/O port |
| `inw(port)` | Read 16-bit word from I/O port |
| `inl(port)` | Read 32-bit dword from I/O port |
| `io_wait()` | Write 0 to port 0x80 (delay) |

## Interrupt Control

| Function | Description |
|---|---|
| `cli()` | Disable interrupts |
| `sti()` | Enable interrupts |
| `hlt()` | Halt CPU until next interrupt |
| `cpu_relax()` | `pause` instruction (spin-loop hint) |
| `cpu_halt()` | Disable interrupts, halt forever (NORETURN) |

## MSR Access

| Function | Description |
|---|---|
| `rdmsr(msr)` | Read 64-bit MSR |
| `wrmsr(msr, val)` | Write 64-bit MSR |

## Control Register Access

| Function | Description |
|---|---|
| `read_cr0()` / `write_cr0(val)` | CR0 (cache control, write protect) |
| `read_cr2()` | CR2 (page fault linear address) |
| `read_cr3()` / `write_cr3(val)` | CR3 (page table base) |
| `read_cr4()` / `write_cr4(val)` | CR4 (SMEP, PGE, etc.) |

## IRQ Save/Restore

| Function | Description |
|---|---|
| `irq_save()` | Save RFLAGS, disable interrupts, return saved flags |
| `irq_restore(flags)` | Restore RFLAGS to saved value |

## Data Structures

### `cpu_state_t` (Interrupt/Syscall Frame)

184-byte packed structure pushed by ISR stubs:

| Offset | Field | Description |
|---|---|---|
| 0x00-0x38 | r15-r8 | General purpose registers |
| 0x40-0x78 | rbp, rdi, rsi, rdx, rcx, rbx, rax | More GPRs |
| 0x80 | int_no | Interrupt vector number |
| 0x88 | error_code | CPU error code (or 0) |
| 0x90 | rip | Return instruction pointer |
| 0x98 | cs | Code segment |
| 0xA0 | rflags | Flags register |
| 0xA8 | rsp | Stack pointer |
| 0xB0 | ss | Stack segment |

### `gdt_entry_t` (8 bytes, packed)

| Field | Size | Description |
|---|---|---|
| limit_low | u16 | Segment limit bits 0-15 |
| base_low | u16 | Base address bits 0-15 |
| base_mid | u8 | Base address bits 16-23 |
| access | u8 | Access byte |
| granularity | u8 | Granularity + flags + limit bits 16-19 |
| base_high | u8 | Base address bits 24-31 |

### `idt_entry_t` (16 bytes, packed)

| Field | Size | Description |
|---|---|---|
| offset_low | u16 | Handler offset bits 0-15 |
| selector | u16 | Code segment selector |
| ist | u8 | Interrupt Stack Table index |
| type_attr | u8 | Gate type + DPL + present bit |
| offset_mid | u16 | Handler offset bits 16-31 |
| offset_high | u32 | Handler offset bits 32-63 |
| zero | u32 | Reserved (must be zero) |

Last reviewed: 2026-07-22
