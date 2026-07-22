# Contributing Overview

This document provides an overview of the contribution workflow for the Kyronix kernel. It is the child of [Contributing](index.md).

## Workflow

- The project uses a standard fork-and-pull workflow.
- All code must pass `make fmt-check` (clang-format) before submission.
- The kernel is written in C11 with the project's `.clang-format` style.
- Assembly files use AT&T syntax with Intel intrinsics.
- The project uses Kconfig for kernel configuration.

Last reviewed: 2026-07-22
