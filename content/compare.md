+++
title = "Compare compilers"
description = "Side-by-side comparison of Julia's compilation workflows, runtime footprints, language coverage, and real-world trade-offs in 2026."
wide = true
+++

Julia offers a smooth trade-off curve between **full dynamic language flexibility** on one end
and **tiny, zero-runtime native binaries** on the other. Use the matrix below to match a tool
to your exact deployment constraints.

## Capabilities matrix

| Workflow / Tool | Output artifacts | Runtime bundled | Typical small binary / bundle | Startup | GC & stdlibs | Dynamic dispatch / `eval` | Multi-lib isolation | Min Julia | Status (2026) |
|-----------------|------------------|-----------------|-------------------------------|---------|--------------|---------------------------|---------------------|-----------|---------------|
| **REPL / JIT** + `PrecompileTools.jl` | Native `pkgimages` (`.so` / `.ji`) | Full local Julia | Cached in depot | ~0.2–1 s | Full | Full | N/A | Any (1.13 fastest) | Production |
| **`Pkg.Apps`** | CLI shims in `~/.julia/bin` | Local Julia install | Source + pkgimage | ~0.2–1 s | Full | Full | Isolated env | 1.12+ | Production |
| **`JuliaC.jl` (untrimmed)** | `.exe`, `.so`/`.dylib`/`.dll`, sysimage, `.o`, `.bc` | Full `libjulia` bundle | ~150–400 MB bundle | Fast | Full | Full | Yes (`--privatize`) | 1.12+ | Active / Evolving |
| **`JuliaC.jl` (`--trim=safe`)** | `.exe`, `.so`/`.dylib`/`.dll`, `.o`, `.bc` | **Trimmed** `libjulia` + needed JLLs | **~1–2 MB exe** (+ ~50–90 MB support bundle; ~60–300 MB for complex libs) | Very fast (~10s ms) | Full GC, **pruned** stdlibs | **Rejected** at compile time (safe) | Yes (`--privatize`) | 1.12+ (1.13 best) | Experimental → rapidly maturing |
| **`PackageCompiler.jl`** (`create_sysimage`, `create_app`, `create_library`) | Sysimage, relocatable app, C shared lib | Full `libjulia` + JLLs | ~150–600+ MB bundle | Fast | Full | Full | Single runtime | 1.10+ | Production stable |
| **`StaticCompiler.jl`** + `StaticTools.jl` | Native `.exe`, C shared lib (`.so`/`.dylib`/`.dll`) | **None** (`libc`/`libm` only) | **~8–200 KB** | Instant (<1 ms) | **None** (no GC, no stdlibs) | None | Native | 1.10+ | Experimental / Niche |
| **`StaticRuntime.jl`** | Single native binary | Minimal pure-Julia runtime (no `libjulia`) | Sub-MB to few MB | Instant | Optional minimal pure-Julia GC | Restricted | Native | 1.11+ | Experimental (2025–2026) |
| **`WasmTarget.jl` / `WasmCodegen.jl`** | Browser / WASI `.wasm` (WasmGC) | Browser WasmGC (no `libjulia`) | ~10s–100s KB `.wasm` | Instant in browser | Browser WasmGC | Closed-world subset | Per-module sandbox | 1.12+ | Experimental (2025–2026) |
| **`Reactant.jl`** + `ReactantServer.jl` | XLA executables, StableHLO bundles, gRPC server | XLA / PJRT runtime | Compiled XLA / StableHLO | Fast after compile | Traced arrays (`RArray`) | Traced out at compile time | Service / module | 1.10+ | Active production ML |
| **`GPUCompiler.jl`** (`CUDA.jl`, `AMDGPU.jl`, `Metal.jl`, `oneAPI.jl`, `cuTile.jl`) | PTX, AMDGCN, AIR, SPIR-V, Tile IR | Vendor GPU driver | Kernel device code | Microseconds launch | No GC in device kernels | None in kernels | Per-device | 1.10+ | Production |
| **`Symbolics.jl` (`CTarget`)** | Standalone `.c`, `.stan`, `.m` source code | **None** | Pure C source | Instant | N/A (symbolic math) | N/A | Native | 1.10+ | Production |

## Binary footprint & build-time benchmarks (2025–2026)

Representative numbers reported across JuliaCon 2025–2026 benchmarks, `JuliaC.jl` case
studies, and Model-Based Engineering evaluations:

| Workload | `PackageCompiler.jl` (default) | `JuliaC.jl` (`--trim=safe`, Julia 1.12/1.13) | `StaticCompiler.jl` |
|----------|--------------------------------|---------------------------------------------|---------------------|
| Minimal "Hello World" CLI | ~120–150 MB bundle | **~0.9–1.7 MB executable** (+ ~90 MB portable bundle) | **~8–30 KB** (no runtime) |
| Model-Based Engineering FMU (`FMIExport.jl` / Dyad) | Hundreds of MB, lengthy build | **~58–59 MB**, up to **~10× faster** build | N/A (requires runtime) |
| High-Energy Physics C-ABI library (`JetReconstruction.jl`) | ~400–600 MB, 20–40 min build | **~300 MB** self-contained bundle, <10 min build | N/A (idiomatic packages) |
| Pure-Julia FFT shared library (`FFTL.jl`, Nov 2026) | Large full-runtime lib | Compact trimmed `libfftl` exposing FFTW3 C/Fortran ABI | Possible for fixed-size kernels only |

## Quick decision guide

- **Developing locally or sharing a Julia package?**
  Stay in the **REPL / JIT** with `Revise.jl` and ship `PrecompileTools.jl` workloads. Use
  `PackageCompiler.create_sysimage` only if startup latency remains a bottleneck for heavy
  environments.
- **Shipping a CLI tool to Julia developers?**
  Declare `function (@main)(args)` and publish a **`Pkg` App** (Julia 1.12+).
- **Shipping a standalone executable or C shared library to non-Julia users?**
  Start with **`JuliaC.jl` (`--trim=safe`)** on Julia 1.12/1.13+. If your codebase relies on
  heavily dynamic packages that block `--trim=safe`, either migrate dynamic sites (using
  `JET.jl` / `DispatchDoctor.jl`), test `--trim=unsafe-warn`, or fall back to untrimmed
  `JuliaC.jl` / **`PackageCompiler.jl`**.
- **Distributing a Julia library to Python or C users automatically?**
  Use **`JuliaLibWrapping.jl` + `JLWInterop`** (Julia 1.13+) on top of `JuliaC.jl` for
  self-contained packages, or **`PythonCall.jl` / `JuliaCall`** when live dynamic access to
  Julia is preferred.
- **Loading multiple independent Julia plugins into one C/C++ host (e.g. a DAW)?**
  Compile with **`JuliaC.jl --output-lib --privatize`** (or use `StaticCompiler.jl` if no GC
  is needed).
- **Packaging a desktop GUI app for macOS, Windows, and Linux?**
  Combine **`PackageCompiler.jl`** (or trimmed `JuliaC.jl`) with **`AppBundler.jl`** and
  **`RelocatableFolders.jl`**.
- **Targeting bare-metal, real-time audio/control, or ultra-tiny C/Fortran micro-libraries?**
  Use **`StaticCompiler.jl` + `StaticTools.jl`** (or watch **`StaticRuntime.jl`**), verified
  with **`AllocCheck.jl`** and **`Bumper.jl`**.
- **Targeting web browsers with zero server backend?**
  Use **`WasmTarget.jl`** (with `Therapy.jl` / `Snapshot.jl`) or **`WasmCodegen.jl`**.
- **Targeting TPUs, multi-GPU ML, or whole-program AD + inference serving?**
  Use **`Reactant.jl`** + **`Enzyme.jl`** + **`ReactantServer.jl`**.
