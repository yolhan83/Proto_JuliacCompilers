+++
title = "JuliaLibWrapping.jl: Automatic C and Python bindings on top of JuliaC"
date = 2026-09-22
[params]
author = "JuliaCompilers Contributors"
+++

With Julia 1.13 and `JuliaC.jl --export-abi`, `JuliaLibWrapping.jl` (JLW) and `JLWInterop`
automate turning Julia packages into standalone C libraries and installable Python packages
with NumPy integration and native exception translation—without requiring Julia on user machines.

<!--more-->

Shipping high-performance Julia algorithms to Python and C teams used to mean choosing
between asking every end-user to install a Julia runtime (via `JuliaCall` / `PyJulia`) or
hand-writing brittle C shim code, headers, and `ctypes` wrappers around `PackageCompiler.jl`.

[**`JuliaLibWrapping.jl`**](/bindings/julialibwrapping/) removes that boilerplate by splitting
the workflow into two pieces:

- **`JLWInterop`**: A lightweight, dependency-free runtime package providing standard ABI
  layout types (`CArray`, `CString`, `JLWStatus`) and the `@api` macro.
- **`JuliaLibWrapping`**: A build-time wrapper generator that runs `JuliaC.jl` with trimming
  and `--export-abi`, reads the emitted JSON ABI description, and generates C `.h` headers and
  complete installable Python packages (`ctypes` + NumPy + native Python exceptions) with the
  trimmed runtime bundled inside.

```julia
using JLWInterop

scale(a::Vector{Float64}; factor::Float64 = 2.0) = factor .* a
@api scale(a::Vector{Float64}; factor::Float64 = 2.0)::Vector{Float64}
```

R and MATLAB wrapper generators are actively being built on the same ABI foundation. Read the
full walkthrough on the [`JuliaLibWrapping.jl` page](/bindings/julialibwrapping/).
