+++
title = "JuliaC.jl & Trimming"
description = "Julia 1.12+ ahead-of-time compiler driver for standalone executables, C-ABI shared libraries, object files, dead-code trimming (--trim), runtime privatization, and ABI export."
weight = 2
badge = "Julia 1.12+"
badgeNew = true

[[params.packages]]
name = "JuliaC.jl"
repository = "https://github.com/JuliaLang/JuliaC.jl"
documentation = "https://github.com/JuliaLang/JuliaC.jl#readme"
+++

[`JuliaC.jl`](https://github.com/JuliaLang/JuliaC.jl) provides the official `juliac`
ahead-of-time (AOT) compiler CLI and programmatic Julia API introduced alongside **Julia 1.12**
and refined through **Julia 1.13**. It streamlines turning Julia code or packages into compact
native executables, C-compatible shared libraries, custom system images, object files, or LLVM
bitcode through an explicit three-stage pipeline: **compile → link → bundle**.

> **When to use `JuliaC.jl`:** Choose `JuliaC.jl` when you want to distribute standalone
> executables or shared libraries to users without Julia installed—especially when binary
> footprint, fast startup, C-ABI interoperability, or multiple-library coexistence matter.

## Installation & requirements

- **Julia 1.12+** (Julia 1.13+ recommended for improved trimming diagnostics, faster builds,
  and [`JuliaLibWrapping.jl`](/bindings/julialibwrapping/) integration)
- A native C linker/compiler (`clang` or `gcc` on Linux/macOS; MSYS2 MinGW on Windows)

Install `juliac` as a native Julia App (or add `JuliaC` to a build environment):

```julia-repl
pkg> app add JuliaC

$ export PATH="$HOME/.julia/bin:$PATH"
$ juliac --help
```

## All output targets

`JuliaC.jl` is not limited to executables—it supports five distinct artifact targets:

| Flag | Output artifact | Typical entry point & use case |
|------|-----------------|--------------------------------|
| `--output-exe <name>` | Standalone native executable | Requires `function (@main)(args::Vector{String})::Cint` (or `Base.Experimental.entrypoint`) |
| `--output-lib <path>` | Shared library (`.so`, `.dylib`, `.dll`) | Exports `Base.@ccallable` functions for C, C++, Python, Rust, Fortran, R, MATLAB |
| `--output-sysimage <path>` | Custom system image | Precompiled sysimage usable with `julia --sysimage` |
| `--output-o <path>` | Native object file (`.o` / `.obj`) | Custom external linking with CMake, Bazel, or firmware linkers |
| `--output-bc <path>` | LLVM bitcode (`.bc`) | Cross-language LTO or custom LLVM pass pipelines |

## Trimming (`--trim`) vs. untrimmed builds

Historically, compiling Julia apps meant shipping the full standard library, compiler, and
metadata—leading to 150–500+ MB bundles. Julia 1.12 introduced semantic **code trimming**,
which starts from declared entry points (`@main`, `@ccallable`, `Base.Experimental.entrypoint`),
traces reachable typed code, and strips unreachable IR, metadata, and unused stdlibs.

| Mode | Behavior | Trade-off |
|------|----------|-----------|
| *(default / `--trim=no`)* | Full runtime & compiler retained; no reachability pruning | Supports arbitrary dynamic dispatch & `eval`, larger bundles |
| `--trim=safe` | Strict static verification; **rejects** unverified dynamic dispatch | Smallest binaries (~1–2 MB exe + compact runtime bundle); guarantees no missing methods at runtime |
| `--trim=unsafe-warn` | Trims aggressively and warns on dynamic sites | Useful while migrating a package toward `--trim=safe` |
| `--trim=unsafe` | Trims without failing on dynamic sites | Small output, but uninferred runtime paths may throw |

> **Tip for `--trim=safe`:** Static inference drives trimming. Packages in the SciML and
> type-stable numerical ecosystems compile cleanly, while heavily dynamic APIs (such as
> untyped `DataFrames` pipelines) may need type-stable alternatives like `StructArrays.jl` or
> `TypedTables.jl`. Use [`JET.jl`](https://github.com/aviatesk/JET.jl) and
> [`DispatchDoctor.jl`](https://github.com/MilesCranmer/DispatchDoctor.jl) to catch dynamic
> dispatches early.

## Building a trimmed standalone executable

Define a package with a `@main` entry point:

```julia
module MyCLI

function (@main)(args::Vector{String})::Cint
    println(Core.stdout, "Hello from trimmed Julia! Args: ", length(args))
    return 0
end

end
```

Compile and bundle it into a relocatable directory:

```sh
juliac path/to/MyCLI \
    --output-exe mycli \
    --bundle build \
    --trim=safe \
    --experimental
```

The resulting `build/` directory contains the trimmed native executable, pruned runtime
libraries (`libjulia` and needed JLL artifacts), and rewritten rpaths (`@bundle`) ready to
copy to another machine of the same OS and CPU architecture.

## Building shared libraries, ABI export & privatization

For libraries embedded into C, C++, Python, or host applications, `JuliaC.jl` provides three
critical production capabilities:

1. **`Base.@ccallable` exports (`--compile-ccallable` / `add_ccallables = true`)**: Exposes
   typed Julia functions with C calling conventions.
2. **`--export-abi`**: Emits structured JSON ABI metadata and C-compatible ABI declarations
   describing exported entry points and types—consumed directly by automatic wrapper generators
   like [`JuliaLibWrapping.jl`](/bindings/julialibwrapping/).
3. **`--privatize` (`BundleRecipe(privatize = true)`)**: On Linux and macOS, salts/renames the
   bundled runtime library symbols (such as `libjulia`) and rewrites binary imports. This solves
   the **multiple-runtime conflict**: a single host process (like an audio DAW loading multiple
   plugins, or a C++ application loading independent Julia modules) can safely load several
   compiled Julia shared libraries side-by-side without symbol collisions.

```julia
module FastMathLib

Base.@ccallable function poly_eval(x::Cdouble)::Cdouble
    return 3.0 * x^2 + 2.0 * x + 1.0
end

end
```

```sh
juliac path/to/FastMathLib \
    --output-lib libfastmath \
    --trim=safe \
    --experimental \
    --export-abi fastmath_abi.json \
    --privatize \
    --bundle dist/
```

## Programmatic Recipe API (`ImageRecipe`, `LinkRecipe`, `BundleRecipe`)

Rather than shelling out to the CLI, build scripts and CI pipelines can drive `JuliaC.jl`
directly from Julia using its three-stage recipe structs:

```julia
using JuliaC

img = ImageRecipe(
    output_type    = "--output-lib",   # or "--output-exe", "--output-sysimage"
    file           = "path/to/FastMathLib",
    trim_mode      = "safe",
    add_ccallables = true,
    verbose        = true,
)

link = LinkRecipe(
    image_recipe = img,
    outname      = "dist/libfastmath",
    rpath        = "@bundle",
)

bun = BundleRecipe(
    link_recipe = link,
    output_dir  = "dist",
    privatize   = true,
)

compile_products(img)
link_products(link)
bundle_products(bun)
```

## Ecosystem & real-world adoption

- **[`JuliaLibWrapping.jl`](/bindings/julialibwrapping/)**: Consumes `juliac --export-abi`
  output to automatically generate C headers and pip-installable Python (`ctypes` + NumPy)
  packages.
- **[`FFTL.jl`](/showcases/#fftljl-fftw3-compatible-shared-library-via-juliac)**: Compiles a
  pure-Julia FFT library with `juliac --trim=safe` into an FFTW3-compatible C/Fortran shared
  library.
- **Model-Based Engineering & Embedded Targets**: Used with
  [`FMIExport.jl` / `FMIBuild.jl`](/compilers/symbolic-fmi/) and Dyad to produce smaller,
  faster-building FMUs and deploy type-stable control loops to ARM and RISC-V targets.
