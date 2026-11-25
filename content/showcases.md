+++
title = "Showcases"
description = "Real-world 2025–2026 deployments demonstrating trimmed JuliaC libraries, automatic Python/C bindings, embedded FMUs, zero-server WebAssembly notebooks, TPU MLIR serving, and compiler-powered IDEs."
+++

The projects below illustrate how Julia's 2025–2026 compilation stack is used in production
and research—spanning drop-in C/Fortran numerical libraries, CERN high-energy physics,
industrial control systems, browser-native notebooks, and accelerator serving.

## FFTL.jl: FFTW3-compatible shared library via `JuliaC`

Announced in **November 2026**, **FFTL.jl** (*Fastest Fourier Transform in Lübeck*) is a
pure-Julia Fast Fourier Transform implementation that compiles with **`JuliaC.jl` and
`--trim=safe`** (Julia 1.12+) into a standalone shared library (`libfftl`) exposing an
**FFTW3-compatible C and Fortran interface**—without writing a single line of C.

- **Key technique**: Strictly type-stable pure-Julia FFT kernels + `Base.@ccallable` entry
  points + `juliac --output-lib --trim=safe`.
- **Why it matters**: Demonstrates that Julia can now deliver drop-in, binary-compatible
  numerical libraries directly into existing C, C++, and Fortran HPC codebases.

## JetReconstruction.jl: High-Energy Physics C++ integration

In **High-Energy Physics (HEP)** benchmarks presented across JuliaCon 2025–2026 and CERN
studies, [`JetReconstruction.jl`](https://github.com/JuliaHEP/JetReconstruction.jl) evaluated
compiling idiomatic Julia jet-clustering algorithms into C-ABI shared libraries via `JuliaC.jl`
and `PackageCompiler.jl` for integration into C++ reconstruction frameworks:

- Once JIT warm-up overhead is eliminated through AOT compilation, the Julia shared library
  **matches or exceeds hand-tuned C++ (FastJet)** performance.
- Moving from legacy `PackageCompiler.jl` bundles (~400–600 MB, 20–40 min builds) to
  **`JuliaC.jl` with trimming** cut build times under 10 minutes and trimmed self-contained
  bundles substantially.

## Dyad & `FMIExport.jl` / `FMIBuild.jl`: Trimmed FMUs & embedded controls

Model-Based Engineering workflows—including JuliaHub's **Dyad** platform and the open-source
[`FMIExport.jl`](https://github.com/ThummeTo/FMIExport.jl) /
[`FMIBuild.jl`](https://github.com/ThummeTo/FMIBuild.jl) stack—export Julia differential
equation and control models into industry-standard **Functional Mock-up Units (`.fmu`)** and
embedded **ARM / RISC-V** binaries:

- Adopting **`JuliaC.jl`** + trimming reduced FMU export build times by up to **~10×** and
  dropped archive sizes from hundreds of megabytes down to **~58–59 MB**.
- Pairing `JuliaC` with **[`AllocCheck.jl`](https://github.com/JuliaLang/AllocCheck.jl)**
  verifies zero-allocation real-time control loops prior to embedded deployment.

## `JuliaLibWrapping.jl`: Zero-Julia Python & C scientific packages

Introduced in **September 2026**, [`JuliaLibWrapping.jl`](https://github.com/JuliaInterop/JuliaLibWrapping.jl)
turns a Julia package into a self-contained **Python** (`ctypes` + NumPy) package and **C**
shared library + header using a single `@api` annotation on Julia 1.13+:

```julia
using JLWInterop

scale(a::Vector{Float64}; factor::Float64 = 2.0) = factor .* a
@api scale(a::Vector{Float64}; factor::Float64 = 2.0)::Vector{Float64}
```

End-users install and import the generated Python package with native NumPy arrays and native
Python exceptions—without ever installing Julia or managing a Julia project environment.

## `Snapshot.jl` & `Therapy.jl`: Zero-server Pluto notebooks & Wasm islands

Built on **[`WasmTarget.jl`](https://github.com/GroupTherapyOrg/WasmTarget.jl)** in 2026:

- **[`Snapshot.jl`](https://snapshot.show/)** exports interactive Pluto.jl notebooks into
  static HTML + WebAssembly pages. Sliders, plots, and Julia numerical cells execute directly
  in the visitor's browser with zero backend server costs.
- **[`Therapy.jl`](https://github.com/GroupTherapyOrg/Therapy.jl)** brings an islands
  architecture to Julia web apps, compiling `@island` interactive UI components directly to
  WebAssembly with fine-grained reactive signals.

## `Reactant.jl` + `Lux.jl` + `ReactantServer.jl`: MLIR/XLA on GPUs, TPUs & gRPC

Using **[`Reactant.jl`](https://github.com/EnzymeAD/Reactant.jl)** with
[`Lux.jl`](https://lux.csail.mit.edu/) and **`EnzymeMLIR`**, Julia machine-learning and
scientific-ML models compile to **MLIR / StableHLO** and run via **XLA** across CPUs, NVIDIA
and AMD GPUs, and Google Cloud TPUs. Compiled StableHLO artifacts export cleanly to
**`ReactantServer.jl`** for production **KServe V2** gRPC serving without a Python runtime.

## `JETLS.jl` + `JuliaLowering.jl`: Compiler-powered IDE intelligence

Showcased at JuliaCon 2025–2026, **[`JETLS.jl`](https://github.com/aviatesk/JETLS.jl)**
reimagines Julia IDE support by running code through Julia's **actual type-inference compiler**
and **[`JuliaLowering.jl`](https://github.com/JuliaLang/JuliaLowering.jl)**—bringing
macro-aware definitions, accurate provenance, and live type-sensitive diagnostics to VS Code
and Zed.
