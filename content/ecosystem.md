+++
title = "Ecosystem directory"
description = "Complete reference of 45+ Julia packages for JIT tuning, AOT binaries, trimming, static no-GC compilation, WebAssembly, MLIR/XLA, GPU compilers, introspection, and automatic bindings."
wide = true
+++

This directory catalogs packages across the entire Julia compilation and interoperability
landscape, highlighting capabilities added or matured across **2025–2026**.

## 1. Ahead-of-Time binaries, libraries & desktop bundling

| Package | Repository | Real capabilities & notes |
|---------|------------|---------------------------|
| **`JuliaC.jl`** | [JuliaLang/JuliaC.jl](https://github.com/JuliaLang/JuliaC.jl) | Official Julia 1.12+ AOT CLI (`juliac`, installable via `pkg> app add JuliaC`) and 3-stage recipe API (`ImageRecipe`, `LinkRecipe`, `BundleRecipe`). Emits `--output-exe`, `--output-lib`, `--output-sysimage`, `--output-o`, `--output-bc`; supports `--trim=safe/unsafe-warn/unsafe/no`, `--bundle`, `--export-abi`, and `--privatize` for multi-runtime coexistence. |
| **`PackageCompiler.jl`** | [JuliaLang/PackageCompiler.jl](https://github.com/JuliaLang/PackageCompiler.jl) | Production-stable `create_sysimage` (incremental/non-incremental), `create_app`, and `create_library`. Preserves full dynamic Julia; supports `precompile_execution_file`, `precompile_statements_file`, `filter_stdlibs`, `cpu_target`, `header_files`, `include_lazy_artifacts`, and `audit`. |
| **`AppBundler.jl`** | [PeaceFounder/AppBundler.jl](https://github.com/PeaceFounder/AppBundler.jl) | Packages Julia GUI and desktop apps into native OS installers: macOS `.app`/`.dmg`, Windows `MSIX`, and Linux `Snap`, with staging, code-signing, sandboxing, and modern trimmed-build support. |
| **`RelocatableFolders.jl`** | [JuliaPackaging/RelocatableFolders.jl](https://github.com/JuliaPackaging/RelocatableFolders.jl) | `@path` macro keeping data assets safely relocatable across precompilation, `PackageCompiler.jl`, and `JuliaC.jl` bundles. |
| **`Pkg.Apps`** *(built-in)* | [JuliaLang/Pkg.jl](https://github.com/JuliaLang/Pkg.jl) | Julia 1.12+ native CLI app installation (`pkg> app add`) exposing `Base.@main` entry points as isolated shims in `~/.julia/bin`. |

## 2. Zero-runtime, no-GC static & low-allocation compilation

| Package | Repository | Real capabilities & notes |
|---------|------------|---------------------------|
| **`StaticCompiler.jl`** | [tshort/StaticCompiler.jl](https://github.com/tshort/StaticCompiler.jl) | Built on `GPUCompiler.jl` + LLVM; compiles a strict, type-stable, GC-free subset of Julia to tiny KB-scale standalone executables (`compile_executable`) and shared libraries (`compile_shlib`) with zero `libjulia` dependency. |
| **`StaticTools.jl`** | [brenhinkeller/StaticTools.jl](https://github.com/brenhinkeller/StaticTools.jl) | GC-free companion types (`MallocArray`, `MallocMatrix`, `StackArray`, `StaticString`, `MallocString`) and libc I/O/memory wrappers for `StaticCompiler.jl`. |
| **`StaticRuntime.jl`** *(2025–2026)* | [brenhinkeller/StaticRuntime.jl](https://github.com/brenhinkeller/StaticRuntime.jl) | Experimental layer atop `StaticCompiler.jl` using `@device_override`, LLVM IR passes, and a minimal pure-Julia runtime (with optional lightweight GC & I/O) to compile broader idiomatic Julia into a single no-`libjulia` binary. |
| **`Bumper.jl`** | [MasonProtter/Bumper.jl](https://github.com/MasonProtter/Bumper.jl) | Thread-local bump/arena allocator (`@no_escape`, `@alloc`) eliminating GC heap allocations for dynamically sized temporary buffers. |
| **`AllocArrays.jl`** | [ericphanson/AllocArrays.jl](https://github.com/ericphanson/AllocArrays.jl) | `AllocArray` and `CheckedAllocArray` redirecting `similar` calls to a dynamically scoped `Bumper.jl` allocator so non-bumper-aware packages avoid GC. |
| **`AllocCheck.jl`** | [JuliaLang/AllocCheck.jl](https://github.com/JuliaLang/AllocCheck.jl) | `@check_allocs` statically analyzes optimized LLVM IR to prove functions perform zero heap allocations or dynamic dispatches. |

## 3. Automatic bindings & cross-language interoperability

| Package | Repository | Direction & capabilities |
|---------|------------|--------------------------|
| **`JuliaLibWrapping.jl` + `JLWInterop`** *(Sept 2026)* | [JuliaInterop/JuliaLibWrapping.jl](https://github.com/JuliaInterop/JuliaLibWrapping.jl) | **Julia → C & Python (AOT, no Julia install)** (Julia 1.13+). Consumes `JuliaC --export-abi` metadata + `@api` macro & dependency-free `JLWInterop` (`CArray`, `CString`, `JLWStatus`) to auto-generate C headers and pip-installable Python `ctypes`/NumPy packages with native Python exceptions. R & MATLAB generators in progress. |
| **`PythonCall.jl` / `JuliaCall`** | [JuliaPy/PythonCall.jl](https://github.com/JuliaPy/PythonCall.jl) | **Python ⇄ Julia (live runtime)**. Symmetric zero-copy NumPy/Arrow/Pandas sharing, automatic env setup via `juliapkg`, active Python 3.11–3.14 & free-threaded build work. |
| **`jlrs`** | [Taaitaaiger/jlrs](https://github.com/Taaitaaiger/jlrs) | **Rust ⇄ Julia**. Safe Rust crate embedding sync/async Julia with `ndarray` support and GC rooting, plus `julia_module!` macro exporting Rust structs/functions directly into Julia. |
| **`CxxWrap.jl`** | [JuliaInterop/CxxWrap.jl](https://github.com/JuliaInterop/CxxWrap.jl) | **C++ → Julia**. Boost.Python-style C++ wrapper library mapping C++ classes, templates, smart pointers, and functions into Julia. |
| **`WrapIt`** | [grasph/wrapit](https://github.com/grasph/wrapit) | **Auto C++ → `CxxWrap.jl`**. Clang/LLVM-based generator that parses massive C++ headers (e.g. ROOT, Geant4, HepMC3) and automatically emits `CxxWrap` bindings. |
| **`Clang.jl`** | [JuliaInterop/Clang.jl](https://github.com/JuliaInterop/Clang.jl) | **Auto C → Julia**. LLVM/libclang header parser automatically generating clean static `@ccall` wrappers, structs, and enums. |
| **`CBinding.jl`** | [analytech-solutions/CBinding.jl](https://github.com/analytech-solutions/CBinding.jl) | **Auto C → Julia**. Parses C headers directly into zero-overhead Julia types matching exact C ABI layouts (unions, bitfields, enums, alignment). |
| **`JuliaConnectoR`** | [stefan-m-lenz/JuliaConnectoR](https://github.com/stefan-m-lenz/JuliaConnectoR) | **R → Julia**. Functional R package calling Julia packages over a binary TCP/IP protocol with automatic data translation. |
| **`RCall.jl`** | [JuliaInterop/RCall.jl](https://github.com/JuliaInterop/RCall.jl) | **Julia → R**. Embeds R in Julia with interactive `$` REPL mode and data-frame interop. |
| **`MATLAB.jl`** | [JuliaInterop/MATLAB.jl](https://github.com/JuliaInterop/MATLAB.jl) | **Julia → MATLAB**. Invokes MATLAB engine and exchanges matrices directly from Julia. |
| **`JavaCall.jl`** | [JuliaInterop/JavaCall.jl](https://github.com/JuliaInterop/JavaCall.jl) | **Julia → Java/JVM**. Loads JVM classes and invokes Java methods from Julia via JNI. |
| **`BinaryBuilder.jl`** | [JuliaPackaging/BinaryBuilder.jl](https://github.com/JuliaPackaging/BinaryBuilder.jl) | **Foreign → Julia JLLs**. Cross-compiles C, C++, Rust, and Fortran libraries into portable JLL binary artifacts automatically bundled by `JuliaC.jl` and `PackageCompiler.jl`. |

## 4. WebAssembly & browser compilation (2025–2026)

| Package | Repository | Real capabilities & notes |
|---------|------------|---------------------------|
| **`WasmTarget.jl`** *(April 2026)* | [GroupTherapyOrg/WasmTarget.jl](https://github.com/GroupTherapyOrg/WasmTarget.jl) | Registered Julia-to-WebAssembly compiler target (`GPUCompiler.jl` / `wasm32-unknown-wasi` & inferred IR) compiling Julia functions and web islands to `.wasm`. |
| **`WasmCodegen.jl`** *(2026)* | [gbaraldi/WasmCodegen.jl](https://github.com/gbaraldi/WasmCodegen.jl) | Closed-world WasmGC compiler modeled after `juliac --trim`; traces reachable inferred IR, serializes Julia objects, and compiles solvers/parsers directly to browser WasmGC without `libjulia`. |
| **`WebAssemblyCompiler.jl`** | [tshort/WebAssemblyCompiler.jl](https://github.com/tshort/WebAssemblyCompiler.jl) | Foundational Julia-to-WasmGC compiler translating `code_typed` IR into browser-runnable WasmGC bytecode. |
| **`WebAssemblyInterfaces.jl`** | [tshort/WebAssemblyInterfaces.jl](https://github.com/tshort/WebAssemblyInterfaces.jl) | Generates JavaScript type definitions and shared-memory bindings (`wasm-ffi`) bridging JS and Julia-compiled WebAssembly modules. |
| **`Therapy.jl`** *(2026)* | [GroupTherapyOrg/Therapy.jl](https://github.com/GroupTherapyOrg/Therapy.jl) | Document-first "islands architecture" full-stack Julia web framework compiling `@island` components to WebAssembly with reactive signals. |
| **`Snapshot.jl`** *(July 2026)* | [snapshot.show](https://snapshot.show/) | Exports interactive Pluto.jl notebooks to static HTML + WebAssembly via `WasmTarget.jl`—zero Julia server required. |

## 5. MLIR/XLA, GPU, AD & symbolic/model compilers

| Package | Repository | Real capabilities & notes |
|---------|------------|---------------------------|
| **`Reactant.jl` & `ReactantServer.jl`** | [EnzymeAD/Reactant.jl](https://github.com/EnzymeAD/Reactant.jl) | Traces Julia to MLIR/StableHLO and compiles via XLA across CPUs, NVIDIA/AMD GPUs, Google TPUs, and experimental accelerators; built-in `EnzymeMLIR` AD, `@compile` AOT, StableHLO export, and KServe V2 gRPC serving. |
| **`Enzyme.jl`** | [EnzymeAD/Enzyme.jl](https://github.com/EnzymeAD/Enzyme.jl) | LLVM-level forward/reverse automatic differentiation directly on optimized LLVM IR (including mutation, GPU kernels, and C/C++ calls). |
| **`GPUCompiler.jl`** | [JuliaGPU/GPUCompiler.jl](https://github.com/JuliaGPU/GPUCompiler.jl) | Reusable LLVM/IR compiler framework powering `CUDA.jl`, `AMDGPU.jl`, `Metal.jl`, `oneAPI.jl`, `OpenCL.jl`, `StaticCompiler.jl`, and `WasmTarget.jl`. |
| **`LLVM.jl`** | [maleadt/LLVM.jl](https://github.com/maleadt/LLVM.jl) | High-level Julia interface to LLVM IR building, optimization passes, target codegen, and ORC JIT. |
| **`cuTile.jl`** *(2026)* | [JuliaGPU/cuTile.jl](https://github.com/JuliaGPU/cuTile.jl) | Tile-based GPU compiler lowering Julia array fragments to NVIDIA Tile IR for Tensor Cores and Tensor Memory Accelerators. |
| **`KernelAbstractions.jl` & `AcceleratedKernels.jl`** | [JuliaGPU/KernelAbstractions.jl](https://github.com/JuliaGPU/KernelAbstractions.jl) | Portable `@kernel` compiler & parallel primitives targeting CUDA, ROCm, Metal, oneAPI, OpenCL, and CPU threads. |
| **`SPIRV.jl`** | [serenity4/SPIRV.jl](https://github.com/serenity4/SPIRV.jl) | Pure-Julia toolkit for reading, transforming, and emitting SPIR-V shader/compute binaries. |
| **`Symbolics.jl` & `ModelingToolkit.jl`** | [JuliaSymbolics/Symbolics.jl](https://github.com/JuliaSymbolics/Symbolics.jl) | `build_function` compiles symbolic graphs to standalone Julia, **C** (`CTarget`), **Stan** (`StanTarget`), or **MATLAB** (`MATLABTarget`) source code. |
| **`FMIExport.jl` & `FMIBuild.jl`** | [ThummeTo/FMIExport.jl](https://github.com/ThummeTo/FMIExport.jl) | Exports Julia models to standard Functional Mock-up Units (`.fmu`), increasingly accelerated by `JuliaC.jl` for ~10× faster builds and compact archives. |

## 6. JIT latency, static analysis, LSP & compiler internals

| Package | Repository | Real capabilities & notes |
|---------|------------|---------------------------|
| **`PrecompileTools.jl`** | [JuliaLang/PrecompileTools.jl](https://github.com/JuliaLang/PrecompileTools.jl) | `@setup_workload` / `@compile_workload` caching native machine code into `pkgimages` during package precompilation to eliminate TTFX. |
| **`SnoopCompile.jl`** | [JuliaDebug/SnoopCompile.jl](https://github.com/JuliaDebug/SnoopCompile.jl) | Profiles compiler inference bottlenecks (`@snoop_inference`) and diagnoses method invalidations (`@snoop_invalidations`). |
| **`JET.jl`** | [aviatesk/JET.jl](https://github.com/aviatesk/JET.jl) | Abstract-interpretation static analyzer (`@report_call`, `@report_opt`) catching dynamic dispatch, method errors, and optimization hazards. |
| **`JETLS.jl`** *(2025–2026)* | [aviatesk/JETLS.jl](https://github.com/aviatesk/JETLS.jl) | Next-generation compiler-powered Language Server Protocol (VS Code & Zed) backed by Julia's real type inference and `JuliaLowering.jl`. |
| **`Cthulhu.jl`** | [JuliaDebug/Cthulhu.jl](https://github.com/JuliaDebug/Cthulhu.jl) | Interactive `@descend` compiler explorer stepping recursively through typed Julia IR, effects, LLVM IR, and native assembly. |
| **`DispatchDoctor.jl`** | [MilesCranmer/DispatchDoctor.jl](https://github.com/MilesCranmer/DispatchDoctor.jl) | `@stable` macro enforcing strict type stability during development and CI, throwing `TypeInstabilityError` on accidental dynamic dispatch. |
| **`Revise.jl`** | [timholy/Revise.jl](https://github.com/timholy/Revise.jl) | Hot-reloads edited package and script code in a live REPL so JIT compilation costs are paid once per session. |
| **`DaemonMode.jl`** | [dmolina/DaemonMode.jl](https://github.com/dmolina/DaemonMode.jl) | Persistent background Julia daemon executing short CLI scripts with near-zero startup latency. |
| **`BaseCompiler.jl` (`Compiler` stdlib)** *(Julia 1.12+)* | [JuliaLang/BaseCompiler.jl](https://github.com/JuliaLang/BaseCompiler.jl) | Exposes Julia's compiler as a Pkg-switchable standard library for upgrading or experimenting with custom compiler branches per project. |
| **`JuliaSyntax.jl` & `JuliaLowering.jl`** *(2025–2026)* | [JuliaLang/JuliaLowering.jl](https://github.com/JuliaLang/JuliaLowering.jl) | Pure-Julia parser and lowering rewrite delivering exact source provenance, hygienic macros, and better diagnostics across the compiler and IDEs. |
| **`CassetteOverlay.jl`** | [JuliaDebug/CassetteOverlay.jl](https://github.com/JuliaDebug/CassetteOverlay.jl) | Lightweight contextual dispatch and method overlays built on `Base.Experimental.@overlay` and `OverlayMethodTable`. |
| **`IRTools.jl`** | [FluxML/IRTools.jl](https://github.com/FluxML/IRTools.jl) | High-level SSA IR inspection and transformation toolkit for compiler passes and source-to-source experiments. |
