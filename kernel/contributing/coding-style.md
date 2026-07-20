# Coding Style

Kyronix uses a consistent coding style across the entire kernel codebase. All contributions must adhere to these conventions.

## Formatter

The project uses `clang-format` with LLVM style and the following overrides:

- Indent width: 4 spaces
- Column limit: 100

Format your changes before committing:

```bash
make fmt
```

## C Language Conventions

- **Standard:** C11 (`-std=c11`)
- **Naming:** `snake_case` for functions, variables, and types. `UPPER_CASE` for macros and constants.
- **Type prefixes:** Use descriptive prefixes for struct types: `proc_t`, `vfs_node_t`, `vmm_space_t`, `pci_dev_t`.
- **Header guards:** Use `#pragma once`.
- **Includes:** System headers in `<>`, project headers in `""`. Order: own header first, then project headers, then system headers.
- **Static assertions:** Use `_Static_assert` to validate struct layouts at compile time.

## File Organization

- Each source file should be responsible for exactly one subsystem or component.
- Headers declare interfaces; `.c` files implement them.
- Assembly files (`.S`) are used for architecture-specific code (trampolines, context switch, syscall entry).

## Error Handling

- Functions that can fail return `int` (0 on success, negative errno on failure) or pointer (`NULL` on failure).
- Use standard Linux errno values (`EINVAL`, `ENOMEM`, `ENOENT`, etc.).
- Always check return values of allocation functions (`pmm_alloc`, `kmalloc`, etc.).

## Memory Safety

- Always validate user pointers before dereferencing them.
- Use `uptr_ok_w()` / `uptr_ok_r()` for user pointer validation.
- Free resources on error paths (goto-based cleanup or inline cleanup).

> **Warning:** Never use `memcpy` with user pointers directly. Always validate the range first with `vmm_user_range_ok()` or `vmm_user_range_fault_in()`.
