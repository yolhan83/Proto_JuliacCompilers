+++
title = "Get started"
description = "Go from zero to compiling Julia—whether you want fast REPL iteration, a trimmed executable, a C/Python shared library, a zero-GC static library, WebAssembly, or MLIR/XLA."
+++

This guide walks you through picking the right compilation workflow, installing the tools,
building your first compiled artifact, and diagnosing type-stability or trimming issues.
Where code differs across tools, the examples on this page have a tab per workflow: **pick
yours in any block, and every other example on the page follows automatically.**

## 1. Pick your compilation workflow

| If you want to… | Start with | Min Julia |
|-----------------|------------|-----------|
| Develop interactively and cut package TTFX | [REPL / JIT + `PrecompileTools.jl`](/compilers/jit-repl/) | Any (1.12/1.13 best) |
| Ship a CLI tool to users who have Julia | `Pkg.Apps` (`pkg> app add`) | 1.12+ |
| Ship a small standalone executable or C shared library | [`JuliaC.jl` (`--trim=safe`)](/compilers/juliac/) | 1.12+ |
| Ship a local sysimage or full dynamic desktop app | [`PackageCompiler.jl` + `AppBundler.jl`](/compilers/packagecompiler/) | 1.10+ |
| Build a tiny KB-scale library with zero `libjulia` or GC | [`StaticCompiler.jl` + `StaticTools.jl`](/compilers/static-compiler/) | 1.10+ |
| Auto-generate a pip-installable Python or C package | [`JuliaLibWrapping.jl` + `JLWInterop`](/bindings/julialibwrapping/) | 1.13+ |
| Run Julia interactively in web browsers without a server | [`WasmTarget.jl` / `WasmCodegen.jl`](/compilers/webassembly/) | 1.12+ |
| Compile numerical / ML code to CPUs, GPUs, and TPUs | [`Reactant.jl` (MLIR / XLA)](/compilers/accelerators-mlir/) | 1.10+ |

## 2. Install the toolchain

Install the packages for your chosen workflow (press `]` in the Julia REPL to enter `pkg>`
mode):

{{< workflows >}}
{{< workflow "REPL / JIT" >}}
```julia-repl
pkg> add PrecompileTools JET Cthulhu DispatchDoctor Revise
```
{{< /workflow >}}
{{< workflow "JuliaC.jl" >}}
```julia-repl
pkg> app add JuliaC

$ export PATH="$HOME/.julia/bin:$PATH"
$ juliac --help
```
{{< /workflow >}}
{{< workflow "PackageCompiler.jl" >}}
```julia-repl
pkg> add PackageCompiler RelocatableFolders
```
{{< /workflow >}}
{{< workflow "StaticCompiler.jl" >}}
```julia-repl
pkg> add StaticCompiler StaticTools Bumper AllocCheck
```
{{< /workflow >}}
{{< workflow "JuliaLibWrapping.jl" >}}
```julia-repl
pkg> app add JuliaC
pkg> add JuliaLibWrapping JLWInterop
```
{{< /workflow >}}
{{< workflow "WebAssembly" >}}
```julia-repl
pkg> add WasmTarget
# Or for closed-world WasmGC codegen:
pkg> add https://github.com/gbaraldi/WasmCodegen.jl
```
{{< /workflow >}}
{{< workflow "Reactant.jl" >}}
```julia-repl
pkg> add Reactant Enzyme
```
{{< /workflow >}}
{{< /workflows >}}

> **Native C compiler requirement:** `JuliaC.jl`, `PackageCompiler.jl`, and `StaticCompiler.jl`
> invoke a system C linker/compiler (`clang` or `gcc` on Linux/macOS; MSYS2 MinGW on Windows).

## 3. Write your entry point

Every compilation mode uses ordinary Julia functions—only the entry-point annotation or type
constraints differ:

{{< workflows >}}
{{< workflow "REPL / JIT" >}}
```julia
module MyPkg

using PrecompileTools

export rms
rms(x::AbstractVector{Float64}) = sqrt(sum(abs2, x) / length(x))

@setup_workload begin
    xs = Float64[1.0, 2.0, 3.0]
    @compile_workload begin
        rms(xs)
    end
end

end
```
{{< /workflow >}}
{{< workflow "JuliaC.jl" >}}
```julia
module MyApp

# Entry point for `--output-exe`:
function (@main)(args::Vector{String})::Cint
    println(Core.stdout, "Args received: ", length(args))
    return 0
end

# Entry point for `--output-lib`:
Base.@ccallable function rms_c(ptr::Ptr{Cdouble}, n::Csize_t)::Cdouble
    s = 0.0
    for i in 1:Int(n)
        v = unsafe_load(ptr, i)
        s += v * v
    end
    return sqrt(s / n)
end

end
```
{{< /workflow >}}
{{< workflow "PackageCompiler.jl" >}}
```julia
module MyApp

function julia_main()::Cint
    println("Hello from PackageCompiler! ARGS = ", ARGS)
    return 0
end

end
```
{{< /workflow >}}
{{< workflow "StaticCompiler.jl" >}}
```julia
using StaticCompiler, StaticTools

function rms_static(x::MallocVector{Float64})::Float64
    s = 0.0
    @inbounds for i in eachindex(x)
        s += x[i] * x[i]
    end
    return sqrt(s / length(x))
end
```
{{< /workflow >}}
{{< workflow "JuliaLibWrapping.jl" >}}
```julia
module MyStats

using JLWInterop

rms(x::Vector{Float64}) = sqrt(sum(abs2, x) / length(x))

@api rms(x::Vector{Float64})::Float64

end
```
{{< /workflow >}}
{{< workflow "WebAssembly" >}}
```julia
# Type-stable, pure-Julia function compiled to WasmGC
function rms_loop(n::Int)::Float64
    s = 0.0
    for i in 1:n
        v = Float64(i)
        s += v * v
    end
    return sqrt(s / n)
end
```
{{< /workflow >}}
{{< workflow "Reactant.jl" >}}
```julia
using Reactant

rms(x) = sqrt(sum(abs2, x) / length(x))

x = Reactant.to_rarray(Float32[1.0, 2.0, 3.0, 4.0])
```
{{< /workflow >}}
{{< /workflows >}}

## 4. Compile & run

{{< workflows >}}
{{< workflow "REPL / JIT" >}}
```julia-repl
julia> using MyPkg, JET

julia> @report_opt rms(Float64[1.0, 2.0, 3.0])
No errors detected

julia> rms(Float64[1.0, 2.0, 3.0])
2.160246899469287
```
{{< /workflow >}}
{{< workflow "JuliaC.jl" >}}
```julia-repl
$ juliac path/to/MyApp --output-exe myapp --trim=safe --experimental --bundle build/

$ ./build/bin/myapp hello world
Args received: 2
```
{{< /workflow >}}
{{< workflow "PackageCompiler.jl" >}}
```julia-repl
julia> using PackageCompiler

julia> create_app("path/to/MyApp", "build/MyAppCompiled"; filter_stdlibs=true)

$ ./build/MyAppCompiled/bin/MyApp hello world
Hello from PackageCompiler! ARGS = ["hello", "world"]
```
{{< /workflow >}}
{{< workflow "StaticCompiler.jl" >}}
```julia-repl
julia> compile_shlib(rms_static, (MallocVector{Float64},), "./build", "librms")

$ ls -lh build/librms.*
# Standalone kilobyte-scale shared library with zero libjulia dependency
```
{{< /workflow >}}
{{< workflow "JuliaLibWrapping.jl" >}}
```julia-repl
julia> using JuliaLibWrapping

julia> wrap_library("path/to/MyStats", "dist/"; targets=[:c, :python])

>>> import numpy as np, mystats
>>> mystats.rms(np.array([1.0, 2.0, 3.0]))
2.160246899469287
```
{{< /workflow >}}
{{< workflow "WebAssembly" >}}
```julia-repl
julia> using WasmCodegen

julia> compile_wasm("rms.wasm", rms_loop, (Int,))
# Load `rms.wasm` directly in Chrome, Firefox, Safari, or Node (WasmGC enabled)
```
{{< /workflow >}}
{{< workflow "Reactant.jl" >}}
```julia-repl
julia> compiled_rms = @compile rms(x)

julia> compiled_rms(x)
2.738613f0
```
{{< /workflow >}}
{{< /workflows >}}

## 5. Making code trim-safe & allocation-free

Almost every compilation failure in `JuliaC --trim=safe`, `StaticCompiler`, `WasmCodegen`, or
GPU kernels traces back to **type instability** or **unintended heap allocations**. Four tools
let you find and fix them systematically during normal REPL development:

1. **`JET.jl` (`@report_opt`, `@report_call`)**: Finds dynamic dispatches, boxed captured
   variables, and unresolved method calls statically.
2. **`DispatchDoctor.jl` (`@stable`)**: Annotate functions or entire modules so your test suite
   throws `TypeInstabilityError` the moment a dynamic dispatch is introduced.
3. **`Cthulhu.jl` (`@descend`)**: Step interactively down the call tree to see exactly which
   line turned concrete types into `Any` or a large `Union`.
4. **`AllocCheck.jl` (`@check_allocs`)**: Verifies at LLVM IR level that a hot loop or
   real-time callback triggers zero heap allocations.

## Going further

- **[Side-by-side comparison](/compare/)**: Compare runtime dependencies, binary sizes,
  startup latency, GC support, and dynamic-dispatch trade-offs across every tool.
- **[Bindings & add-ons](/bindings/)**: Generate automatic Python packages, C headers, Rust
  crates, FMUs, and JavaScript interfaces.
- **[Complete ecosystem directory](/ecosystem/)**: Browse all 45+ compilation, introspection,
  and interoperability packages.
- **[Showcases](/showcases/)**: See real-world 2025–2026 deployments from `FFTL.jl`, CERN
  high-energy physics, Model-Based Engineering, browser Wasm notebooks, and TPU inference.
