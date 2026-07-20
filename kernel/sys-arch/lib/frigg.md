# Frigg

This document describes the frigg library used in the Kyronix ecosystem.

## Overview

Frigg is a C++ utility library providing common data structures and helpers. It is used by the Managarm-derived components of the system (servers, drivers).

## Components

Frigg typically provides:

- Smart pointers (`UniquePtr`, `SharedPtr`)
- Smart handles (`Handle`)
- Callback utilities
- Utility types (`Optional`, `Bitset`)
- String utilities

> **Note:** Frigg is a dependency inherited from the Managarm project. The Kyronix kernel itself is written in C and does not directly use frigg.
