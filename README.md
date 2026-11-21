# Parallel Ray Tracing with OpenMP

An academic C++17 project exploring parallel ray tracing, race conditions, synchronization techniques, and performance evaluation using OpenMP.

**Contributor:** Ameera 
**Course:** ARTI 503 — Parallel Computer Architecture

## Overview

This project extends a sequential ray tracer by distributing image rendering across multiple CPU threads. It investigates how concurrent updates to a shared pixel counter can cause race conditions and evaluates four OpenMP synchronization techniques to prevent lost updates.

## Contributions

- Parallelized the rendering loop using `#pragma omp parallel for schedule(static)`.
- Investigated race conditions affecting the shared `pixels_done` counter.
- Implemented and evaluated Critical, Atomic, Reduction, and Lock synchronization during development.
- Used a thread-local random number generator (`std::mt19937`) initialized with seed 42.
- Measured execution time and verified completed-pixel counts across thread configurations.

## Repository Scope

The source code published here represents the **Lock implementation**. Critical, Atomic, Reduction, and the unsynchronized version were tested during development; their separate source snapshots are not included. The results below document those experiments.

## Experimental Setup

| Parameter | Value |
|---|---|
| Image resolution | 400 × 225 pixels |
| Samples per pixel | 10 |
| Maximum ray depth | 20 |
| Initial RNG seed | 42 |
| Parallel thread counts | 2, 4, 8 |
| Repetitions | 5 per configuration |
| Processor | Intel Core i7-1195G7 |
| RAM | 16 GB |
| Operating system | Windows |
| Compiler | GCC 16.2.0 (MSYS2 UCRT64) |
| Optimization | `-O3` |

## Synchronization Techniques

| Technique | Approach |
|---|---|
| Critical | Protects the counter increment with a critical section. |
| Atomic | Performs the shared counter increment atomically. |
| Reduction | Combines thread-private counters after the parallel loop. |
| Lock | Acquires and releases an explicit OpenMP lock around the counter increment. |

## Experimental Results

Average execution times in seconds, based on five runs per configuration:

| Technique | 2 threads | 4 threads | 8 threads |
|---|---:|---:|---:|
| Critical | 2.38388 | 1.60587 | **1.28212** |
| Atomic | 2.63227 | 1.71457 | 1.72591 |
| Reduction | 2.55014 | 1.71416 | 1.66828 |
| Lock | 2.34609 | 1.67347 | 1.77911 |

All 60 synchronized test runs reported the expected count of 90,000 completed pixels. The unsynchronized version produced incorrect pixel counts. Critical achieved the lowest observed average at eight threads on the test laptop.

These results establish counter correctness for the tested configurations, not complete program verification or exact image equivalence across thread counts. Each thread owns its RNG, with no explicit per-pixel or per-row reseeding during rendering.

## Build and Run

### Requirements

- A C++17 compiler with OpenMP support
- Windows with MSYS2 UCRT64 GCC, or another compatible C++/OpenMP toolchain

### Windows (PowerShell)

Open a terminal in the directory containing `main.cc`. If using the default MSYS2 installation, add the compiler to the PATH when needed:

```powershell
$env:Path = "C:\msys64\ucrt64\bin;" + $env:Path
```

Compile the included Lock implementation:

```powershell
g++ -std=c++17 -O3 -fopenmp main.cc -o raytracer.exe
```

Run with eight threads and save the image:

```powershell
$env:OMP_NUM_THREADS = "8"
cmd /c ".\raytracer.exe > image.ppm"
```

Change `OMP_NUM_THREADS` to `2` or `4` to test other configurations. Rendering time and completed-pixel count are printed to the terminal. Image output is written to `image.ppm` and excluded from the measured rendering time.

## Key Source Files

| File | Purpose |
|---|---|
| `main.cc` | Scene construction, camera configuration, and program entry point |
| `camera.h` | Parallel rendering, pixel counter, and Lock synchronization |
| `rtweekend.h` | Random number generation and common utilities |
| `hittable.h`, `hittable_list.h`, `sphere.h` | Geometry and ray intersections |
| `material.h` | Material scattering behavior |
| `ray.h`, `vec3.h`, `interval.h`, `color.h` | Ray, vector, interval, and color utilities |
| `COPYING.txt` | Original project license |

## Acknowledgments and License

The underlying ray tracing code is based on Peter Shirley's *Ray Tracing in One Weekend*. The original code is dedicated to the public domain under CC0 1.0. Original copyright and license notices are retained in the source files and `COPYING.txt`.

This repository documents an academic extension focused on OpenMP parallelization, synchronization, and performance analysis.
