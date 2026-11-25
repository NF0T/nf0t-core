# nf0t-core

Application-independent amateur radio logbook code, written in C++23 with Qt 6.
It is a static library shared by the NF0T applications:

- **[NF0T Logger](https://github.com/NF0T/NF0T-Logger)**, a general-purpose QSO logger.
- **CC Companion**, a 3905 Century Club logger (in development).

The code was extracted from NF0T Logger with its history intact. The reasons
for the split, and what was moved, are in NF0T Logger's
[`docs/nf0t-core-plan.md`](https://github.com/NF0T/NF0T-Logger/blob/develop/docs/nf0t-core-plan.md).

## What's in it

| Area | Contents |
|---|---|
| `src/core/logbook` | `Qso`, the central QSO value type, and query filters |
| `src/core/adif` | ADIF v3 parser (length-based, spec-compliant) and writer |
| `src/core` | `Callsign` (ITU validation, prefix/suffix parsing), `Maidenhead` (grid ↔ lat/lon, distance), `BandPlan` |
| `src/database` | `DatabaseInterface`, with SQLite and MariaDB backends. This is NF0T Logger's storage; other consumers own their own schema |
| `src/lookup` | Callsign lookup providers: QRZ XML Data API and offline CTY.dat |

The library never reads application settings. Consumers pass configuration in
(`CtyDatConfig`, `QrzXmlConfig`, `AdifWriterOptions`).

## Using it

Add it as a git submodule and include it from CMake:

```cmake
add_subdirectory(libs/nf0t-core)
target_link_libraries(my-app PRIVATE nf0t-core)
```

Pin the submodule to a release tag, never to an untagged commit.

## Building on its own

Requirements: CMake ≥ 3.20, a C++23 compiler (GCC 13+, Clang 16+, MSVC 2022+),
and Qt 6 (Core, Sql, Network, Test) with the SQLite SQL driver.

```bash
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build --parallel
ctest --test-dir build --output-on-failure
```

Pass `-DNF0T_CORE_BUILD_TESTS=OFF` to skip the tests. A consumer that includes
the library with `add_subdirectory` gets the tests registered with its own
`ctest` unless it sets that option.

## Versioning

Tags use CalVer, matching NF0T Logger: `vYY.MM.W`, where `W` is the
Sunday–Saturday week of the month. A second tag in the same week adds a fourth
segment (`v26.11.4.1`).

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
