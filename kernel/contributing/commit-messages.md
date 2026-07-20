# Commit Messages

Kyronix follows a structured commit message format for clarity and automated changelog generation.

## Format

```
<type>: <short summary>

<body>

<footer>
```

## Types

| Type | Purpose |
|------|---------|
| `feat` | A new feature or capability |
| `fix` | A bug fix |
| `docs` | Documentation-only changes |
| `style` | Code style changes (formatting, no logic change) |
| `refactor` | Code restructuring without behavior change |
| `test` | Adding or updating tests |
| `build` | Build system or dependency changes |
| `ci` | CI/CD configuration changes |
| `perf` | Performance improvements |
| `revert` | Reverting a previous commit |

## Rules

1. The subject line must be lowercase and imperative mood (e.g., "add", not "added" or "adds").
2. The subject line must not exceed 72 characters.
3. The body should explain *what* and *why*, not *how* (the diff shows how).
4. Reference related issues with `Fixes #123` or `Closes #456` in the footer.

## Examples

```
feat: add timerfd_create syscall

Implement timerfd_create, timerfd_settime, and timerfd_gettime
syscalls for user-space timers backed by kernel timerfd file
descriptors.
```

```
fix: prevent race condition in process reap

The deferred reap mechanism could access a freed process entry
if the reap thread was scheduled before the dying process was
fully cleaned up. Fix by checking PROC_UNUSED before access.
```

> **Tip:** Use `git commit --fixup` to create fixup commits for early review feedback, then `git rebase --autosquash` before merging.
