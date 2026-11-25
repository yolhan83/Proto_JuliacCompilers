+++
title = "REPL, JIT & Precompilation"
description = "Interactive just-ahead-of-time LLVM compilation, native pkgimage caching, TTFX elimination, Pkg Apps, and 2025–2026 compiler-powered tooling."
weight = 1
badge = "Built-in + Tooling"

[[params.packages]]
name = "PrecompileTools.jl"
repository = "https://github.com/JuliaLang/PrecompileTools.jl"
documentation = "https://julialang.github.io/PrecompileTools.jl/stable/"

[[params.packages]]
name = "JET.jl"
repository = "https://github.com/aviatesk/JET.jl"
documentation = "https://aviatesk.github.io/JET.jl/stable/"

[[params.packages]]
name = "JETLS.jl"
repository = "https://github.com/aviatesk/JETLS.jl"
documentation = "https://aviatesk.github.io/JETLS.jl/"

[[params.packages]]
name = "Cthulhu.jl"
repository = "https://github.com/JuliaDebug/Cthulhu.jl"
documentation = "https://juliadebug.github.io/Cthulhu.jl/stable/"
+++

By default, Julia compiles functions **just ahead of time (JAOT)** the first time they are
called with a given tuple of argument types. The pipeline lowers source code into SSA IR,
runs type inference and constant propagation, performs high-level Julia optimizations, and
hands the specialized IR to **LLVM** to emit native machine code.

Understanding and tuning this interactive compiler is the foundation for every other
compilation mode in Julia—especially because the exact same type-inference engine governs
both JIT speed and ahead-of-time trimming.

## Interactive compiler inspection

Julia exposes every stage of its compiler directly from the REPL via `InteractiveUtils`:

```julia-repl
julia> dot2(x, y) = x * x + y * y
dot2 (generic function with 1 method)

julia> @code_lowered dot2(1.0, 2.0)   # Untyped SSA IR after lowering
julia> @code_typed   dot2(1.0, 2.0)   # Type-inferred & optimized Julia IR
julia> @code_warntype dot2(1.0, 2.0)  # Highlights type instabilities in red
julia> @code_llvm    dot2(1.0, 2.0)   # Optimized LLVM IR
julia> @code_native  dot2(1.0, 2.0)   # Target CPU assembly (x86_64 / aarch64)
```

For deep recursive inspection across call chains, [`Cthulhu.jl`](https://github.com/JuliaDebug/Cthulhu.jl)
provides `@descend`, letting you step into any callee, inspect inferred effects, toggle
optimization passes, and switch live between Julia IR, LLVM IR, and native assembly.

## Eliminating latency (TTFX) & precompilation

Since Julia 1.9, package precompilation stores **native machine code** (`pkgimages`) on disk
alongside serialized IR, and **Julia 1.13** accelerated precompilation by roughly 30% while
adding `--trace-eval` for profiling top-level load times.

- **[`PrecompileTools.jl`](https://github.com/JuliaLang/PrecompileTools.jl)**: Package authors
  wrap representative workloads in `@setup_workload` and `@compile_workload` blocks so native
  code is cached at installation time rather than compiled on first user call.
- **[`SnoopCompile.jl`](https://github.com/JuliaDebug/SnoopCompile.jl)**: Profiles what the
  compiler is actually spending time on (`@snoop_inference`) and pinpoints **method
  invalidations** (`@snoop_invalidations`) that silently discard cached native code.
- **[`Revise.jl`](https://github.com/timholy/Revise.jl)**: Keeps one REPL session alive and
  hot-reloads edited functions across packages and scripts (`includet`) without paying startup
  or recompilation latency again.
- **[`DaemonMode.jl`](https://github.com/dmolina/DaemonMode.jl)**: Runs a persistent background
  Julia server so repeated CLI script invocations execute with near-zero startup latency.

```julia
module MyPackage

using PrecompileTools

compute(x::Vector{Float64}) = sum(abs2, x)

@setup_workload begin
    xs = Float64[1.0, 2.0, 3.0]
    @compile_workload begin
        compute(xs)
    end
end

end
```

## Native CLI distribution with `Pkg.Apps` (Julia 1.12+)

Before reaching for full binary bundling, check whether your users already have Julia.
Starting in **Julia 1.12**, `Pkg` natively supports **Apps**: any package declaring a
`function (@main)(args::Vector{String})` entry point and an `[apps]` entry in `Project.toml`
can be installed globally as a command-line shim in `~/.julia/bin`:

```julia-repl
pkg> app add JuliaC

$ juliac --help
```

`Pkg.Apps` gives each CLI tool an isolated environment and cached `pkgimage`—ideal for
developer tools without shipping hundreds of megabytes per binary.

## Type-stability & static analysis tooling

Writing type-stable, allocation-controlled code speeds up JIT execution *and* makes a package
compatible with `JuliaC --trim=safe`, `StaticCompiler`, GPU kernels, and WebAssembly:

| Package | Role | Key capability |
|---------|------|----------------|
| [`JET.jl`](https://github.com/aviatesk/JET.jl) | Static analyzer | `@report_call` & `@report_opt` catch runtime dispatch, method errors, and captured boxes via abstract interpretation |
| [`JETLS.jl`](https://github.com/aviatesk/JETLS.jl) | Compiler-powered LSP (2025–2026) | Replaces approximate syntax-only analysis with Julia's real compiler & `JuliaLowering.jl` in VS Code and Zed |
| [`DispatchDoctor.jl`](https://github.com/MilesCranmer/DispatchDoctor.jl) | Type-stability enforcement | `@stable` throws `TypeInstabilityError` during tests whenever dynamic dispatch sneaks into a function or module |
| [`AllocCheck.jl`](https://github.com/JuliaLang/AllocCheck.jl) | Allocation verification | `@check_allocs` statically analyzes LLVM IR to guarantee zero heap allocations or dynamic dispatches |

## Modular compiler internals (Julia 1.12–1.13+)

Two major 2025–2026 architectural shifts make Julia's compiler itself modular and hackable:

- **[`BaseCompiler.jl` / `Compiler` stdlib](https://github.com/JuliaLang/BaseCompiler.jl)**:
  Starting in Julia 1.12, the compiler is a standard library (`Compiler`) that can be upgraded
  or swapped per project environment directly via `pkg> dev /path/to/Compiler.jl` without
  rebuilding Julia itself.
- **[`JuliaSyntax.jl`](https://github.com/JuliaLang/JuliaSyntax.jl) &
  [`JuliaLowering.jl`](https://github.com/JuliaLang/JuliaLowering.jl)**: Pure-Julia rewrites
  of Julia's parser and lowering passes, delivering accurate source provenance, clean macro
  hygiene, and richer compiler diagnostics powering tools like `JETLS.jl`, `Revise.jl`, and
  `Cthulhu.jl`.
