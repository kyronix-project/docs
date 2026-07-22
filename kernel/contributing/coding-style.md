# Coding Style

This document describes the coding style conventions for the Kyronix kernel. It is the child of [Contributing](index.md).

## Conventions

- **Formatting**: enforced by `clang-format` with the project's `.clang-format` file.
- Run `make fmt` to format all kernel source files.
- Run `make fmt-check` to verify formatting without changes.
- **Naming**: snake_case for functions and variables, UPPER_CASE for macros and constants.
- **Headers**: include what you use, prefer forward declarations.
- **Error handling**: check return values, use goto-based cleanup.
- **Memory**: always check allocation results, free on error paths.
- **Assembly**: AT&T syntax for `.S` files, Intel intrinsics via `<stdint.h>`.

Last reviewed: 2026-07-22
