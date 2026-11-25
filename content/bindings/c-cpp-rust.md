+++
title = "C, C++, Fortran & Rust"
description = "Export Julia as C-ABI shared libraries with JuliaC, PackageCompiler, or StaticCompiler; bridge Rust bidirectionally with jlrs; or auto-wrap C/C++ libraries into Julia with Clang.jl, CBinding.jl, and CxxWrap.jl + WrapIt."
weight = 2
badge = "Systems & FFI"

[[params.packages]]
name = "jlrs"
repository = "https://github.com/Taaitaaiger/jlrs"
documentation = "https://docs.rs/jlrs/latest/jlrs/"

[[params.packages]]
name = "CxxWrap.jl"
repository = "https://github.com/JuliaInterop/CxxWrap.jl"
documentation = "https://github.com/JuliaInterop/CxxWrap.jl#readme"

[[params.packages]]
name = "WrapIt"
repository = "https://github.com/grasph/wrapit"
documentation = "https://github.com/grasph/wrapit#readme"

[[params.packages]]
name = "Clang.jl"
repository = "https://github.com/JuliaInterop/Clang.jl"
documentation = "https://juliainterop.github.io/Clang.jl/stable/"

[[params.packages]]
name = "CBinding.jl"
repository = "https://github.com/analytech-solutions/CBinding.jl"
documentation = "https://github.com/analytech-solutions/CBinding.jl#readme"
+++

Systems languages share the **C Application Binary Interface (ABI)** as a universal bridge.
Depending on which direction you are calling—and whether you can afford a Julia runtime—the
ecosystem provides dedicated compilers and automatic binding generators.

## Exporting Julia to C, C++, and Fortran

Three compilation tools export Julia functions as native shared libraries (`.so`, `.dylib`,
`.dll`) callable from C, C++, and Fortran (`iso_c_binding`):

| Tool | Entry point | Runtime init needed? | Binary / bundle footprint | Multi-lib coexistence |
|------|-------------|----------------------|---------------------------|-----------------------|
| **[`JuliaC.jl`](/compilers/juliac/)** (`--output-lib --trim=safe`) | `Base.@ccallable` | Managed by runtime init | Compact trimmed bundle | Yes via `--privatize` |
| **[`PackageCompiler.jl`](/compilers/packagecompiler/)** (`create_library`) | `Base.@ccallable` | `init_julia()` / `shutdown_julia()` | Full runtime (~150–500 MB) | Single runtime per process |
| **[`StaticCompiler.jl`](/compilers/static-compiler/)** (`compile_shlib`) | Pure GC-free function | **None** | Tiny (~8–200 KB) | Native (no `libjulia`) |

Pair `JuliaC.jl --export-abi` with [`JuliaLibWrapping.jl`](/bindings/julialibwrapping/) to
auto-generate C header files (`.h`) instead of maintaining headers by hand. For pure math
expressions without any Julia runtime at all, [`Symbolics.jl`](/compilers/symbolic-fmi/)
(`build_function(...; target=Symbolics.CTarget())`) emits standalone `.c` source files.

## Rust ⇄ Julia: `jlrs`

[`jlrs`](https://github.com/Taaitaaiger/jlrs) is the premier Rust crate for safe,
high-performance, bi-directional Julia–Rust interoperability:

- **Calling Julia from Rust**: Safely embeds the Julia runtime (sync or async runtime
  backends), invokes arbitrary Julia functions with keyword arguments, catches Julia exceptions
  as Rust `Result`s, and shares N-dimensional array buffers (`ndarray` integration) with
  compile-time GC rooting guarantees.
- **Exporting Rust to Julia**: Provides the `julia_module!` procedural macro to expose Rust
  structs, methods, and functions directly as a native Julia module (paired with `JlrsCore.jl`),
  going far beyond manual `ccall` boilerplate.
- **Zero-runtime Rust → Julia or Julia → Rust**: Rust `cdylib` crates exporting `extern "C"`
  symbols can be called directly via Julia's built-in `@ccall`, and Julia libraries compiled
  with `JuliaC.jl` or `StaticCompiler.jl` can be linked into Rust via standard FFI `extern "C"`
  blocks.

## Auto-generating Julia wrappers for C: `Clang.jl` & `CBinding.jl`

When calling *into* C libraries from Julia, you never need to hand-write `ccall` wrappers for
large headers:

- **[`Clang.jl`](https://github.com/JuliaInterop/Clang.jl)**: Uses LLVM/libclang to parse C
  header files and automatically generate idiomatic, static Julia wrapper files (`@ccall`
  signatures, structs, enums, constants, and function pointers). Combined with
  [`BinaryBuilder.jl`](https://github.com/JuliaPackaging/BinaryBuilder.jl), it is the
  standard pipeline behind hundreds of JLL wrapper packages (such as `MuJoCo.jl`).
- **[`CBinding.jl`](https://github.com/analytech-solutions/CBinding.jl)**: Parses C headers
  directly to construct zero-overhead Julia types matching exact C ABI layouts—including
  unions, bitfields, enums, variadic functions, and compiler-specific alignment.

## Auto-generating Julia wrappers for C++: `CxxWrap.jl` & `WrapIt`

Because C++ has templates, overloading, RAII, and name mangling, wrapping C++ requires a
C++-aware bridge:

- **[`CxxWrap.jl`](https://github.com/JuliaInterop/CxxWrap.jl)**: Inspired by `Boost.Python`
  and `pybind11`. You expose C++ classes, templates (mapped to Julia parametric types),
  smart pointers, and methods in a thin C++ wrapper library (`libcxxwrap-julia`), which Julia
  loads dynamically with native method dispatch.
- **[`WrapIt`](https://github.com/grasph/wrapit)** *(with `WrapIt.jl`)*: A Clang/LLVM-powered
  code generator that **automatically writes `CxxWrap.jl` wrapper code** from large C++
  headers—battle-tested on massive high-energy physics libraries including **ROOT.jl**,
  **Geant4.jl**, and **HepMC3.jl**.

## Companion: `BinaryBuilder.jl` (foreign code → Julia JLLs)

While this site focuses on compiling *Julia* code, [`BinaryBuilder.jl`](https://github.com/JuliaPackaging/BinaryBuilder.jl)
(and next-generation `BinaryBuilder2.jl` efforts) solves the complementary half of the
toolchain: cross-compiling C, C++, Rust, and Fortran libraries into portable, reproducible
binary artifact packages (`*_jll.jl`) across Linux, macOS, Windows, and FreeBSD—which
`JuliaC.jl` and `PackageCompiler.jl` then automatically bundle into standalone Julia builds.
