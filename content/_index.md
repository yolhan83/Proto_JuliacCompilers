+++
title = "JuliaCompilers"

[params]
lead = """
Julia is not just a JIT-compiled language: its typed IR, LLVM pipeline, and modular compiler
infrastructure span interactive REPL sessions, trimmed standalone executables and shared
libraries, custom sysimages, zero-runtime no-GC code, WebAssembly, MLIR/XLA, GPUs, and
automatic foreign-language bindings.
"""

[[params.examples]]
name = "REPL / JIT"
code = """
julia> using PrecompileTools, JET

julia> norm2(x::AbstractVector) = sum(abs2, x)
norm2 (generic function with 1 method)

julia> @report_opt norm2(Float64[1.0, 2.0, 3.0])
No errors detected

julia> @code_native debuginfo=:none norm2(Float64[1.0, 2.0])
        vmovupd (%rsi), %xmm0
        vmulpd  %xmm0, %xmm0, %xmm0
        ...

julia> @compile_workload begin
           norm2(Float64[1.0, 2.0])   # cached into pkgimage
       end
"""

[[params.examples]]
name = "JuliaC.jl"
code = """
pkg> app add JuliaC

$ juliac --output-exe myapp \\
         --trim=safe --experimental \\
         --bundle build/ \\
         path/to/MyApp

# Or compile a C-compatible shared library with isolated runtime:
$ juliac --output-lib libmyalg \\
         --trim=safe --experimental \\
         --export-abi abi.json \\
         --privatize --bundle build/ \\
         path/to/MyLib
"""

[[params.examples]]
name = "PackageCompiler.jl"
code = """
julia> using PackageCompiler

# 1. Local sysimage for instant startup & zero TTFX
julia> create_sysimage([:Plots, :DifferentialEquations];
           sysimage_path="sys_sci.so",
           precompile_execution_file="precompile.jl")

# 2. Relocatable standalone app (full dynamic Julia)
julia> create_app("MyApp", "MyAppCompiled";
           filter_stdlibs=true)

# 3. Relocatable C-callable shared library bundle
julia> create_library("MyLib", "MyLibCompiled";
           lib_name="mylib", header_files=["mylib.h"])
"""

[[params.examples]]
name = "StaticCompiler.jl"
code = """
julia> using StaticCompiler, StaticTools

julia> function fib(n::Int)::Int
           n <= 1 && return n
           a, b = 0, 1
           for _ in 2:n; a, b = b, a + b; end
           return b
       end

# Compiles to a tiny (~10 KB) shared library — NO libjulia, NO GC!
julia> compile_shlib(fib, (Int,), "./", "libfib")

$ ls -lh libfib.dylib
-rwxr-xr-x  14K  libfib.dylib
"""

[[params.examples]]
name = "JuliaLibWrapping.jl"
code = """
# In your Julia library (Julia 1.13+):
julia> using JLWInterop

julia> scale(a::Vector{Float64}; k::Float64=2.0) = k .* a
julia> @api scale(a::Vector{Float64}; k::Float64=2.0)::Vector{Float64}

# Generate C headers + pip-installable Python pkg via JuliaC:
julia> using JuliaLibWrapping
julia> wrap_library("MyLib", "dist/"; targets=[:c, :python])

# End users run in Python — NO Julia install required:
>>> import numpy as np, mylib
>>> mylib.scale(np.array([1.0, 2.0, 3.0]), k=3.0)
array([3., 6., 9.])
"""

[[params.examples]]
name = "WebAssembly"
code = """
# Compile pure Julia to browser WasmGC (WasmCodegen / WasmTarget)
julia> using WasmCodegen

julia> function mandelbrot(cr::Float64, ci::Float64, n::Int)::Int
           zr, zi = 0.0, 0.0
           for k in 1:n
               zr*zr + zi*zi > 4.0 && return k
               zr, zi = zr*zr - zi*zi + cr, 2.0*zr*zi + ci
           end
           return n
       end

julia> compile_wasm("mandel.wasm", mandelbrot, (Float64, Float64, Int))
# Runs directly in browsers with WasmGC — no libjulia needed
"""

[[params.examples]]
name = "Reactant.jl"
code = """
julia> using Reactant

julia> f(x) = sum(sin.(x) .^ 2 .+ cos.(x) .^ 2)

julia> x = Reactant.to_rarray(rand(Float32, 1024, 1024))

# Traces to MLIR / StableHLO and compiles via XLA (CPU/GPU/TPU)
julia> compiled_f = @compile f(x)
julia> compiled_f(x)
1.048576f6
"""
+++

## Choosing the right compilation workflow

Not every Julia project needs the same compilation strategy. In 2025–2026 the ecosystem
converged around clear layers depending on whether you need interactive iteration, small
standalone deliverables, zero-runtime micro-libraries, browser deployment, or foreign-language
bindings:

| Goal | Primary tool | Runtime required? | Typical size |
|------|--------------|-------------------|--------------|
| Interactive development & fast package loads | Built-in JIT + `PrecompileTools.jl` + `Revise.jl` | Full Julia | Pkgimage cache |
| CLI tool for Julia users | `Pkg.Apps` (Julia 1.12+) | Local Julia | Source + cache |
| Small standalone executable or C shared library | [`JuliaC.jl`](/compilers/juliac/) (`--trim=safe`) | Trimmed `libjulia` bundle | ~1–2 MB bin (+ trimmed libs) |
| Automatic Python / C packages (no Julia install) | [`JuliaLibWrapping.jl`](/bindings/julialibwrapping/) + `JuliaC.jl` | Bundled & trimmed | Self-contained wheel / lib |
| Complex dynamic app, local sysimage, or desktop installer | [`PackageCompiler.jl`](/compilers/packagecompiler/) + `AppBundler.jl` | Full `libjulia` bundle | ~150–500+ MB |
| Zero-GC, zero-`libjulia` library or bare-metal kernel | [`StaticCompiler.jl`](/compilers/static-compiler/) + `StaticTools.jl` | None | ~8–200 KB |
| Interactive web app or Pluto notebook in browser | [`WasmTarget.jl` / `WasmCodegen.jl`](/compilers/webassembly/) | Browser WasmGC | ~10s–100s KB `.wasm` |
| ML, AD, and multi-accelerator (CPU / GPU / TPU) | [`Reactant.jl`](/compilers/accelerators-mlir/) + `Enzyme.jl` | XLA / PJRT / gRPC | StableHLO / compiled exe |

See the full [side-by-side comparison](/compare/) or browse all 45+ packages in the
[ecosystem directory](/ecosystem/).

## Community

Compilation, trimming, and interoperability are among the fastest-moving areas in Julia. You
can follow development and ask questions at:

- Julia Discourse: the [Internals & Compiler](https://discourse.julialang.org/c/internals/compiler/24)
  and [Tooling](https://discourse.julialang.org/c/tools/57) categories
- Julia Slack ([register here](https://julialang.org/slack/)): `#static-compilation`,
  `#juliac`, `#internals`, `#compiler-tooling`, `#wasm`, and `#python`
- [JuliaLang/JuliaC.jl](https://github.com/JuliaLang/JuliaC.jl),
  [JuliaLang/PackageCompiler.jl](https://github.com/JuliaLang/PackageCompiler.jl), and
  [JuliaInterop/JuliaLibWrapping.jl](https://github.com/JuliaInterop/JuliaLibWrapping.jl)
  issue trackers and discussions
- Related portal for hardware accelerators: [juliagpu.org](https://juliagpu.org/)
