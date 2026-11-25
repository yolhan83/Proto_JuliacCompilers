+++
title = "StaticCompiler.jl & No-GC Compilation"
description = "True zero-runtime, kilobyte-scale native executables and shared libraries without libjulia or garbage collection, plus StaticRuntime.jl, arena allocators, and zero-allocation checks."
weight = 4
badge = "Zero Runtime"

[[params.packages]]
name = "StaticCompiler.jl"
repository = "https://github.com/tshort/StaticCompiler.jl"
documentation = "https://github.com/tshort/StaticCompiler.jl#readme"

[[params.packages]]
name = "StaticTools.jl"
repository = "https://github.com/brenhinkeller/StaticTools.jl"
documentation = "https://github.com/brenhinkeller/StaticTools.jl#readme"

[[params.packages]]
name = "StaticRuntime.jl"
repository = "https://github.com/brenhinkeller/StaticRuntime.jl"
documentation = "https://github.com/brenhinkeller/StaticRuntime.jl#readme"

[[params.packages]]
name = "Bumper.jl"
repository = "https://github.com/MasonProtter/Bumper.jl"
documentation = "https://github.com/MasonProtter/Bumper.jl#readme"

[[params.packages]]
name = "AllocCheck.jl"
repository = "https://github.com/JuliaLang/AllocCheck.jl"
documentation = "https://julialang.github.io/AllocCheck.jl/stable/"
+++

When even a trimmed `libjulia` runtime is too heavy—for example, in real-time audio DSP,
embedded/bare-metal controllers, micro-kernels called from C/C++/Fortran/Rust without calling
`jl_init()`, or tiny kilobyte-scale CLI utilities—the **StaticCompiler** ecosystem compiles a
strict subset of Julia straight through [`GPUCompiler.jl`](https://github.com/JuliaGPU/GPUCompiler.jl)
and [`LLVM.jl`](https://github.com/maleadt/LLVM.jl) into standalone native machine code.

## `StaticCompiler.jl` + `StaticTools.jl`

[`StaticCompiler.jl`](https://github.com/tshort/StaticCompiler.jl) bypasses `libjulia` and
Julia's garbage collector entirely. Because standard Julia `Array`, `String`, `Dict`, and
exception machinery depend on `libjulia`,
[`StaticTools.jl`](https://github.com/brenhinkeller/StaticTools.jl) supplies drop-in, GC-free
replacements:

- **Memory & collections**: `MallocArray`, `MallocMatrix`, `StackArray`, `MallocString`,
  `StaticString` (`c"..."`)
- **System I/O & libc wrappers**: `printf`, `println` (backed by libc `puts`/`printf`),
  `fopen`/`fclose`, `malloc`/`free`, `InlineString` helpers

### Compiling a zero-runtime shared library (`compile_shlib`)

```julia
using StaticCompiler, StaticTools

function saxpy!(y::MallocVector{Float64}, x::MallocVector{Float64}, a::Float64)
    @inbounds for i in eachindex(y, x)
        y[i] = a * x[i] + y[i]
    end
    return 0
end

# Emits a standalone ~10–30 KB shared library with NO libjulia dependency
compile_shlib(saxpy!, (MallocVector{Float64}, MallocVector{Float64}, Float64), "./", "libsaxpy")
```

### Compiling a standalone native executable (`compile_executable`)

```julia
using StaticCompiler, StaticTools

function hello_static()
    println(c"Hello from zero-runtime Julia!")
    return 0
end

compile_executable(hello_static, (), "./", "hello_static")
```

The resulting binary is typically **8 KB to ~200 KB**, starts in under a millisecond, and has
only libc/libm as dynamic dependencies.

### Constraints to keep in mind

- Strictly **type-stable** and **GC-allocation-free** (no boxed values, no dynamic dispatch, no
  standard `Vector`/`String`/`Dict` heap allocations).
- Standard Julia exceptions and error paths require `@device_override` or explicit bounds/error
  handling (`@inbounds`, status codes).
- Multi-threading via Julia's task scheduler is not available without `libjulia`.

## Bridging to broader Julia: `StaticRuntime.jl` (2025–2026)

[`StaticRuntime.jl`](https://github.com/brenhinkeller/StaticRuntime.jl) is an experimental
next-generation layer built on top of `StaticCompiler.jl` and `GPUCompiler.jl`. Its goal is to
compile much broader, idiomatic Julia programs—including arrays, string handling, and math
algorithms—into a single standalone native binary **still without `libjulia`**, by combining:

- `@device_override` method tables replacing `libjulia`-dependent routines
- Custom LLVM IR transformation passes
- A minimal pure-Julia runtime providing basic I/O and an **optional lightweight pure-Julia
  garbage collector**

## Zero-allocation & arena memory ecosystem

Even outside `StaticCompiler.jl`, writing allocation-free or arena-allocated code is essential
for real-time control systems, embedded ARM/RISC-V targets, and high-frequency numerical loops:

- **[`Bumper.jl`](https://github.com/MasonProtter/Bumper.jl)**: *"Bring Your Own Stack"* —
  thread-local bump/arena allocation. Dynamically sized temporary arrays allocated inside
  `@no_escape` are wiped in O(1) on scope exit with zero GC pressure:

```julia
using Bumper

function fast_kernel(x::Vector{Float64})
    @no_escape begin
        buf = @alloc(Float64, length(x))
        buf .= sin.(x) .+ cos.(x)
        sum(buf)
    end
end
```

- **[`AllocArrays.jl`](https://github.com/ericphanson/AllocArrays.jl)**: Provides `AllocArray`
  and `CheckedAllocArray` wrappers that intercept standard `similar` calls and redirect them to
  a dynamically scoped `Bumper.jl` allocator—allowing existing packages that do not know about
  `Bumper.jl` to run without GC allocations.
- **[`AllocCheck.jl`](https://github.com/JuliaLang/AllocCheck.jl)**: Statically inspects
  optimized LLVM IR via `@check_allocs` to prove that a function performs **zero heap
  allocations** and **zero dynamic dispatches**, reporting exact source locations whenever an
  allocation slips in:

```julia
using AllocCheck

@check_allocs function step!(out::Vector{Float64}, x::Vector{Float64}, dt::Float64)
    @inbounds for i in eachindex(out, x)
        out[i] = x[i] + dt * (x[i] * 0.5)
    end
    return nothing
end
```
