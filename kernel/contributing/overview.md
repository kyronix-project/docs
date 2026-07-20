# Contribution Overview

This section describes the general workflow for contributing to Kyronix.

## Workflow

1. Fork the repository on GitHub.
2. Create a feature branch from `main`.
3. Make your changes, following the [coding style](coding-style.md).
4. Write clear [commit messages](commit-messages.md).
5. Open a pull request against `main`.

## What to Contribute

- **Bug fixes:** If you find a bug, open an issue first, then submit a fix.
- **New features:** Discuss large features in an issue before implementing.
- **Documentation:** Improvements to documentation are always welcome.
- **Tests:** Adding test cases for existing or new functionality is encouraged.

## Code Review

All pull requests undergo code review. The reviewer will check:

- Correctness and safety of the change
- Adherence to the [coding style](coding-style.md)
- Clarity of commit messages
- Absence of regressions (verified via the test suite)

## Testing

Before submitting a pull request, run the test suite:

```bash
make test-iso
make test-run-log
```

This builds a test ISO and runs it in QEMU, executing the built-in test harness. The test harness covers filesystem, process, signal, IPC, timer, I/O, jail, disk, and SMP test phases.

> **Important:** All tests must pass before a pull request will be merged.
