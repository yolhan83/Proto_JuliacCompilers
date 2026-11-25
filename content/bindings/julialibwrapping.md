+++
title = "JuliaLibWrapping.jl (Python & C)"
description = "Automatically turn Julia packages into self-contained shared libraries with generated C headers and pip-installable Python packages (ctypes + NumPy)—no Julia install required."
weight = 1
badge = "Sept 2026"
badgeNew = true

[[params.packages]]
name = "JuliaLibWrapping.jl"
repository = "https://github.com/JuliaInterop/JuliaLibWrapping.jl"
documentation = "https://juliainterop.github.io/JuliaLibWrapping.jl/"

[[params.packages]]
name = "JuliaC.jl"
repository = "https://github.com/JuliaLang/JuliaC.jl"
documentation = "https://github.com/JuliaLang/JuliaC.jl#readme"
+++

Announced in **September 2026** by Tim Holy, Jorge Veyra, and contributors under the
`JuliaInterop` organization, [`JuliaLibWrapping.jl`](https://github.com/JuliaInterop/JuliaLibWrapping.jl)
(**JLW**) closes the gap between compiling a Julia shared library with `JuliaC.jl` and handing
users in **Python** or **C** a polished, native-feeling package.

Unlike dynamic bridges (`JuliaCall` / `PyJulia`) that require end-users to have a local Julia
installation, `JuliaLibWrapping.jl` bundles the compiled library and trimmed runtime directly
inside the generated distribution.

## Architecture: two cleanly separated packages

`JuliaLibWrapping.jl` builds on **`JuliaC.jl`** (specifically its `--export-abi` JSON metadata)
and requires **Julia 1.13+**. It is split into two sub-packages:

1. **`JLWInterop`** *(runtime dependency — zero external dependencies)*: Defines standardized,
   ABI-safe C layout types (`CArray`, `CString`, `JLWStatus`) and the `@api` macro. Package
   authors depend only on `JLWInterop` so their library stays tiny and trimmable.
2. **`JuliaLibWrapping`** *(build-time generator)*: Invokes `JuliaC.jl` to compile and trim
   your package, reads the emitted ABI JSON, and automatically generates:
   - **C**: Clean `.h` header files paired with the compiled shared library (`.so`, `.dylib`,
     `.dll`) and bundled runtime.
   - **Python**: Complete installable Python packages using `ctypes`, with Pythonic façades,
     direct **NumPy** array integration, memory ownership handling, and automatic translation
     of Julia exceptions into native Python exceptions.
   - **R & MATLAB**: Gateway and dynamic-call generator infrastructure is actively under
     development.

## Declaring a foreign API with `@api`

Annotate standard Julia functions with `@api`. JLW automatically generates the `@ccallable`
C-ABI entry points, argument/return conversions, safe exception transport, and ABI metadata:

```julia
module SignalLib

using JLWInterop

function scale_and_shift(a::Vector{Float64}; factor::Float64 = 2.0, offset::Float64 = 0.0)
    if factor < 0
        throw(ArgumentError("factor must be non-negative"))
    end
    return factor .* a .+ offset
end

@api scale_and_shift(
    a::Vector{Float64};
    factor::Float64 = 2.0,
    offset::Float64 = 0.0,
)::Vector{Float64}

end
```

For libraries that must conform to a pre-existing C header specification, manual output
buffers, or bespoke C structs, `JuliaLibWrapping.jl` also supports hand-written ABI entry
points alongside `@api`.

## Generating & consuming C and Python bindings

At build time, run the wrapper generator to compile with `JuliaC.jl` (with trimming enabled)
and emit target-language packages:

```julia
using JuliaLibWrapping

# Compiles via JuliaC, trims unreachable code, bundles runtime, and generates bindings
wrap_library("path/to/SignalLib", "dist/"; targets = [:c, :python])
```

End-users install and import the generated Python package with zero awareness of Julia:

```python
import numpy as np
import signallib

x = np.array([1.0, 2.0, 3.0], dtype=np.float64)
y = signallib.scale_and_shift(x, factor=2.5, offset=1.0)
# array([3.5, 6.0, 8.5])

# Julia ArgumentError propagates cleanly as a native Python exception!
```

## Choosing between `JuliaLibWrapping.jl` and `JuliaCall`

| Criterion | `JuliaLibWrapping.jl` (AOT) | [`JuliaCall` / `PythonCall.jl`](/bindings/python-r-web/#python-pythoncalljl--juliacall--compiled-wheels) (Live) |
|-----------|-----------------------------|-------------------------------------------------------------------------------------------------|
| End-user needs Julia installed? | **No** (self-contained bundle) | Yes (managed automatically via `juliapkg`) |
| API surface | Curated, bounded `@api` functions | Full dynamic access to any Julia package, types, and REPL |
| Startup latency | Fast (AOT compiled & trimmed) | Pays Julia startup & package load latency |
| Multi-library isolation | Supports `JuliaC --privatize` | Single shared Julia runtime in-process |
| Minimum Julia version | Julia 1.13+ | Julia 1.10+ |
