+++
title = "Python, R, MATLAB, Java & Web"
description = "Live-runtime bridges, standalone Python wheels, R and MATLAB connectors, JavaScript/Wasm interfaces, FMUs, and ML inference servers."
weight = 3
badge = "Multi-Language"

[[params.packages]]
name = "PythonCall.jl"
repository = "https://github.com/JuliaPy/PythonCall.jl"
documentation = "https://juliapy.github.io/PythonCall.jl/stable/"

[[params.packages]]
name = "JuliaConnectoR"
repository = "https://github.com/stefan-m-lenz/JuliaConnectoR"
documentation = "https://github.com/stefan-m-lenz/JuliaConnectoR#readme"

[[params.packages]]
name = "RCall.jl"
repository = "https://github.com/JuliaInterop/RCall.jl"
documentation = "https://juliainterop.github.io/RCall.jl/stable/"

[[params.packages]]
name = "MATLAB.jl"
repository = "https://github.com/JuliaInterop/MATLAB.jl"
documentation = "https://github.com/JuliaInterop/MATLAB.jl#readme"

[[params.packages]]
name = "JavaCall.jl"
repository = "https://github.com/JuliaInterop/JavaCall.jl"
documentation = "https://juliainterop.github.io/JavaCall.jl/stable/"
+++

Many scientific and enterprise teams embed Julia inside **Python**, **R**, **MATLAB**,
**Excel**, **Java**, or **browser** workflows. Below are the modern packages covering both
live-runtime bridges and compiled AOT integrations.

## Python: `PythonCall.jl` / `JuliaCall` & compiled wheels

### Live-runtime bridge (`PythonCall.jl` & `JuliaCall`)

[`PythonCall.jl`](https://github.com/JuliaPy/PythonCall.jl) (calling Python from Julia) and
**`JuliaCall`** (`pip install juliacall`, calling Julia from Python) have superseded legacy
`PyCall.jl` / `PyJulia` as the ecosystem standard:

- **Zero-copy buffer sharing**: NumPy arrays, Python buffer-protocol objects, Pandas, and
  Arrow tables wrap directly as Julia arrays (and vice versa) without unnecessary copies.
- **Automatic dependency management**: `juliapkg` manages Julia versions and project
  environments cleanly per Python virtual environment.
- **Modern Python support (2025–2026)**: Tracks Python 3.11 through 3.14, including early
  work for free-threaded Python builds.

```python
# pip install juliacall
from juliacall import Main as jl
import numpy as np

jl.seval("using Statistics")
arr = np.array([1.0, 2.0, 3.0, 4.0])
print(jl.Statistics.mean(arr))   # shares memory with NumPy
```

### Self-contained Python wheels (no Julia installation required)

When end-users should run `pip install mypkg` **without** downloading or managing Julia:
- **Recommended (Julia 1.13+)**: Use **[`JuliaLibWrapping.jl`](/bindings/julialibwrapping/)**
  to compile with `JuliaC.jl --trim=safe --export-abi` and automatically generate a Python
  package with `ctypes`, NumPy integration, and native Python exception translation.
- **Custom wheels**: Compile a shared library with `JuliaC.jl --output-lib --bundle` (or
  `StaticCompiler.compile_shlib` for tiny GC-free kernels) and bundle it inside a Python wheel
  with `setuptools` / `hatch` + `ctypes` or `cffi`.

## R: `JuliaConnectoR` & `RCall.jl`

- **[`JuliaConnectoR`](https://github.com/stefan-m-lenz/JuliaConnectoR)**: CRAN R package for
  calling Julia functions and entire Julia packages from R using a clean functional interface
  over a binary TCP/IP protocol, with automatic translation of vectors, matrices, data frames,
  and structs.
- **[`RCall.jl`](https://github.com/JuliaInterop/RCall.jl)**: Embeds R inside Julia with an
  interactive `$` REPL mode and zero-boilerplate data-frame conversion.
- **Compiled shared libraries**: R's `.C()` and `.Call()` interfaces can load trimmed shared
  libraries produced by `JuliaC.jl` or `StaticCompiler.jl`; native R gateway generation is also
  on the [`JuliaLibWrapping.jl`](/bindings/julialibwrapping/) roadmap.

## MATLAB, Excel & Java

- **MATLAB**:
  - **[`MATLAB.jl`](https://github.com/JuliaInterop/MATLAB.jl)** calls the MATLAB engine from
    Julia.
  - Calling Julia *from* MATLAB works via MATLAB's `loadlibrary` or C MEX wrappers linked
    against a `JuliaC.jl` / `PackageCompiler.jl` / `StaticCompiler.jl` shared library, via
    exported FMUs ([`FMIExport.jl`](/compilers/symbolic-fmi/)), or via
    `Symbolics.MATLABTarget()` for symbolic expressions.
- **Excel**: Compile `@ccallable` routines into a Windows `.dll` with `JuliaC.jl`,
  `PackageCompiler.jl`, or `StaticCompiler.jl` and expose them through a thin C/C++ Excel XLL
  or COM add-in.
- **Java / JVM**: **[`JavaCall.jl`](https://github.com/JuliaInterop/JavaCall.jl)** embeds the
  JVM inside Julia; calling compiled Julia from Java uses JNI or Java's modern Foreign Function
  & Memory (FFM) API against a `JuliaC.jl` shared library.

## Web browsers, simulation standards & ML serving

- **JavaScript / TypeScript ↔ WebAssembly**: Pair [`WasmTarget.jl` or `WasmCodegen.jl`](/compilers/webassembly/)
  with [`WebAssemblyInterfaces.jl`](https://github.com/tshort/WebAssemblyInterfaces.jl) to
  generate typed JavaScript bindings for compiled `.wasm` modules.
- **Simulink / Modelica / ANSYS**: Package Julia models as standard `.fmu` archives with
  [`FMIExport.jl` & `FMIBuild.jl`](/compilers/symbolic-fmi/).
- **Language-agnostic ML & numerical serving**: Compile models to StableHLO with
  [`Reactant.jl`](/compilers/accelerators-mlir/) and serve them over standard **KServe V2**
  gRPC via `ReactantServer.jl` so any language (Go, Python, Java, C++, TypeScript) can call
  compiled Julia models over the network.
