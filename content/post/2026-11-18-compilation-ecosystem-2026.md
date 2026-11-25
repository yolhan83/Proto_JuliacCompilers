+++
title = "State of Julia's compilation ecosystem in 2026"
date = 2026-11-18
[params]
author = "JuliaCompilers Contributors"
+++

From Julia 1.12's `--trim` and `JuliaC.jl` to Julia 1.13's `JuliaLibWrapping.jl`, WasmGC
compilers, `StaticRuntime.jl`, `cuTile.jl`, and `JETLS.jl`, 2025–2026 transformed Julia from
a JIT-first language into a full-spectrum compiler platform.

<!--more-->

For years, deploying Julia outside an interactive REPL meant choosing between two extremes:
either bundling the entire Julia runtime with `PackageCompiler.jl` (full language support,
large footprint) or restricting yourself to a GC-free C-like subset with `StaticCompiler.jl`
(tiny binaries, limited library support).

Across **2025 and 2026**, the middle ground filled in dramatically—and new targets opened up
at both ends of the spectrum:

## 1. Trimmed native binaries & isolated shared libraries (`JuliaC.jl`)

Starting with **Julia 1.12** and accelerated in **Julia 1.13** (~30% faster precompilation and
improved inference diagnostics), [`JuliaC.jl`](/compilers/juliac/) (`pkg> app add JuliaC`)
established a clean *compile → link → bundle* pipeline:

- **`--trim=safe`** strips unreachable IR, metadata, and unused standard libraries, bringing
  minimal executables down to ~1–2 MB (with a pruned runtime support folder).
- **`--export-abi`** emits structured ABI metadata for `@ccallable` functions.
- **`--privatize`** salts bundled `libjulia` symbols on Linux and macOS so multiple
  independently compiled Julia shared libraries can be loaded into the same host process.
- Real-world adoption includes drop-in C/Fortran FFT libraries (`FFTL.jl`), CERN HEP
  reconstruction benchmarks (`JetReconstruction.jl`), and ~10× faster FMU exports
  (`FMIExport.jl` / Dyad).

## 2. Automatic C & Python bindings (`JuliaLibWrapping.jl`)

Announced in **September 2026**, [`JuliaLibWrapping.jl`](/bindings/julialibwrapping/) and its
zero-dependency runtime helper `JLWInterop` read `JuliaC --export-abi` metadata to turn
ordinary `@api`-annotated Julia functions into clean C headers and self-contained,
pip-installable Python packages with `ctypes`, NumPy array support, and native Python
exceptions—requiring **no Julia installation** on the end-user's machine.

## 3. Browser-native WebAssembly (`WasmTarget.jl`, `WasmCodegen.jl`)

Rather than porting `libjulia` via Emscripten, 2026 compilers like
[`WasmTarget.jl`](/compilers/webassembly/) and `WasmCodegen.jl` lower closed-world inferred
Julia IR directly to **WasmGC**. This unlocked [`Therapy.jl`](https://github.com/GroupTherapyOrg/Therapy.jl)
(`@island` interactive web apps) and [`Snapshot.jl`](https://snapshot.show/) (exporting
interactive Pluto notebooks that execute client-side with zero backend server).

## 4. Zero-`libjulia` bridges & low-allocation tooling

[`StaticRuntime.jl`](/compilers/static-compiler/) pushes beyond `StaticCompiler.jl` by pairing
`@device_override` with a minimal pure-Julia runtime and optional lightweight GC, while
`Bumper.jl`, `AllocArrays.jl`, `AllocCheck.jl`, and `DispatchDoctor.jl` make zero-allocation,
type-stable Julia practical across both JIT and AOT builds.

## 5. Accelerators & compiler-powered developer tooling

On the hardware and tooling front, [`Reactant.jl`](/compilers/accelerators-mlir/) brings
MLIR/StableHLO/XLA compilation, `EnzymeMLIR` AD, and KServe V2 gRPC serving (`ReactantServer.jl`)
across CPUs, GPUs, and TPUs; `cuTile.jl` brings NVIDIA Tile IR to Julia; and
[`JETLS.jl`](/compilers/jit-repl/) + `JuliaLowering.jl` + switchable `Compiler` stdlibs bring
real compiler inference straight into VS Code and Zed.

Browse all 45+ tools in the [ecosystem directory](/ecosystem/) or compare trade-offs in the
[comparison matrix](/compare/).
