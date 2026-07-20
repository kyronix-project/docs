# Init

This document describes the init process (PID 1) in Kyronix.

## Overview

The init process is the first user-space process started by the kernel. It is responsible for:

1. Mounting the root filesystem
2. Starting system services
3. Handling orphaned processes
4. Providing the user login interface

## Boot Sequence

When the kernel finishes initialization:

1. Mount the root filesystem (ext2 or CPIO initrd)
2. Search for `/init`, `/sbin/init`, or `/bin/init`
3. Execute the found init binary via `process_exec()`

## User-Space Init

The Kyronix init process (`user/init/init.c`):

1. Sets up the environment
2. Starts the login process (`/bin/login`)
3. Waits for child processes
4. Reaps zombie processes

## Process Reaping

Init is responsible for reaping orphaned processes. When a parent process exits before its children, those children are reparented to PID 1.
