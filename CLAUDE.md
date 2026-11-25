# CLAUDE.md — nf0t-core

Developer guide for Claude Code and human contributors.

## What this is

A static C++23 / Qt 6 library of application-independent logbook code: the
`Qso` type, ADIF v3 parsing and writing, callsign and grid utilities, the band
plan, the SQLite/MariaDB storage backends, and callsign lookup providers. See
[README.md](README.md) for the layout.

Consumers include it as a git submodule at `libs/nf0t-core` and call
`add_subdirectory`. Current consumers: NF0T Logger, and CC Companion (in
development).

## The rule that governs everything else

**The library must never constrain a consumer.** The apps are both QSO
loggers but serve different purposes. Shared code exists where sharing helps
all of them, and no app's needs may be shaped by another's through this
library. In practice:

- **No application coupling.** The library includes only its own sources. There
  is deliberately no include path to any consumer, so a dependency on app code
  is a compile error. Keep it that way.
- **No settings access.** Never read `QSettings` or any app settings singleton.
  Take configuration as a parameter, the way `CtyDatConfig`, `QrzXmlConfig` and
  `AdifWriterOptions` do. Use callables when the value can change at runtime.
- **No shared schema.** `SqlBackendBase` and its migrations are NF0T Logger's
  storage. Other consumers own their own database and migrations. Do not add a
  hook for another app to extend this schema.
- **Additive changes only, where a consumer depends on behaviour.** A change
  for one consumer must leave the others' behaviour unchanged, or be a
  deliberate, tagged break that every consumer adopts on purpose.

## Build and test

```bash
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build --parallel
ctest --test-dir build --output-on-failure
```

CI builds and tests on Linux (Debian Trixie), Windows (MSYS2 UCRT64) and macOS
(Apple Silicon).

## Workflow

- A single `main` branch. Changes arrive by squash-merged PRs.
- A change a consumer needs: PR here → tag → a PR in each consumer that bumps
  the submodule. Never point a submodule at an untagged commit.
- Tags are CalVer `vYY.MM.W` (Sunday–Saturday week of the month), with a fourth
  segment for a second tag in the same week.

## Code conventions

- C++23, `CXX_EXTENSIONS OFF`. No GNU or MSVC extensions.
- No `M_PI`; use `std::numbers::pi`.
- Qt signal/slot syntax: always new-style (`&Class::signal`).
- `QStringLiteral` for all string literals used with Qt APIs.
- Comments explain *why* when it is not obvious, never *what*.
- Every source file starts with the SPDX header:
  `// SPDX-License-Identifier: GPL-3.0-or-later`.

## Database code

- Database operations return `std::expected<T, QString>` and are marked
  `[[nodiscard]]`. Never throw from database code, and never discard a result.
  GCC 14 and Apple clang do not warn on a discarded `std::expected`, so the
  attribute is what makes them warn.
- Schema changes: add a new migration. Never alter an existing one.
- Every migration goes into **both** `SqliteBackend.cpp` and
  `MariaDbBackend.cpp` in the same commit.
