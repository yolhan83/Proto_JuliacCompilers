+++
title = "PackageCompiler.jl & Desktop Bundling"
description = "Battle-tested custom system images, full-runtime relocatable apps, C-callable shared libraries, relocatable assets, and native desktop installers (DMG, MSIX, Snap)."
weight = 3
badge = "Production Stable"

[[params.packages]]
name = "PackageCompiler.jl"
repository = "https://github.com/JuliaLang/PackageCompiler.jl"
documentation = "https://julialang.github.io/PackageCompiler.jl/stable/"

[[params.packages]]
name = "AppBundler.jl"
repository = "https://github.com/PeaceFounder/AppBundler.jl"
documentation = "https://github.com/PeaceFounder/AppBundler.jl#readme"

[[params.packages]]
name = "RelocatableFolders.jl"
repository = "https://github.com/JuliaPackaging/RelocatableFolders.jl"
documentation = "https://github.com/JuliaPackaging/RelocatableFolders.jl#readme"
+++

[`PackageCompiler.jl`](https://github.com/JuliaLang/PackageCompiler.jl) is Julia's mature,
production-proven ahead-of-time builder for **custom system images**, **standalone
applications**, and **C-callable shared libraries**. Unlike strict trimming or static-subset
compilers, `PackageCompiler.jl` bundles the full Julia runtime and compiler—meaning **all**
idiomatic dynamic Julia features, `eval`, reflection, and complex third-party package stacks
work out of the box.

## Three core capabilities of `PackageCompiler.jl`

### 1. Custom system images (`create_sysimage`)

A system image (`.so`, `.dylib`, or `.dll`) bakes loaded packages, global state, and
precompiled native methods directly into the binary Julia loads at startup—eliminating load
time and Time-to-First-X (TTFX) on your local machine or server image:

```julia
using PackageCompiler

create_sysimage(
    [:DataFrames, :CairoMakie, :DifferentialEquations];
    sysimage_path = "sys_analysis.so",
    precompile_execution_file = "precompile_workload.jl",
    incremental = true,     # fast layering on top of Julia's default sysimage
    cpu_target  = "native", # optimize for the local CPU microarchitecture
)
```

Launch Julia with the custom image:

```sh
julia --sysimage sys_analysis.so
```

Key options and nuances:
- **`incremental = true` vs. `false`**: Incremental sysimages build much faster by layering
  onto Julia's base sysimage; `incremental = false` rebuilds from scratch (required if you pass
  `filter_stdlibs = true` to omit unused standard libraries).
- **`precompile_execution_file` & `precompile_statements_file`**: Executes a workload script
  (or replays traced precompile statements from `julia --trace-compile=...`) so hot methods
  are compiled into native code inside the image.
- **Version locking**: Packages baked into a sysimage are pinned to the exact versions active
  at build time until the sysimage is rebuilt.
- **`replace_default = true`**: Optionally replaces Julia's default sysimage (reversible with
  `restore_default_sysimage()`).

### 2. Relocatable standalone applications (`create_app`)

`create_app` turns a Julia package with a `julia_main()::Cint` entry point into a self-contained
directory (executable + custom sysimage + `libjulia` + JLL binary artifacts) that runs on
target machines with the same OS and CPU architecture—no Julia installation required:

```julia
module MyDesktopApp

function julia_main()::Cint
    try
        run_app(ARGS)
    catch
        Base.invokelatest(Base.display_error, Base.catch_stack())
        return 1
    end
    return 0
end

end
```

```julia
using PackageCompiler

create_app(
    "MyDesktopApp",
    "MyDesktopAppCompiled";
    executables = ["myapp" => "julia_main"],
    precompile_execution_file = "precompile_app.jl",
    filter_stdlibs = true,
    include_lazy_artifacts = true,
    include_transitive_dependencies = true,
    cpu_target = "generic;sandybridge,-xsaveopt,clone_all;haswell,-rdrnd,base(1)",
    audit = true,
)
```

### 3. Relocatable C-callable shared libraries (`create_library`)

`create_library` packages a Julia project exporting `Base.@ccallable` functions into a
relocatable folder with `lib/` (the shared library + `libjulia`) and `include/` (your C header
plus `julia_init.h` providing `init_julia` and `shutdown_julia`):

```julia
using PackageCompiler

create_library(
    "MyJuliaLib",
    "MyJuliaLibCompiled";
    lib_name = "myjulialib",
    header_files = ["include/myjulialib.h"],
    precompile_execution_file = "precompile_lib.jl",
    version = v"1.0.0",
    compat_level = "major",
    incremental = false,
    filter_stdlibs = true,
)
```

## Relocatable data & assets: `RelocatableFolders.jl`

A common pitfall when bundling apps is hardcoding `@__DIR__` at precompile time, which breaks
when the compiled app is moved to another machine.
[`RelocatableFolders.jl`](https://github.com/JuliaPackaging/RelocatableFolders.jl) provides
the `@path` macro so asset directories (shaders, fonts, models, configs) resolve transparently
both during development and inside relocated `PackageCompiler.jl` or `JuliaC.jl` bundles:

```julia
using RelocatableFolders

const ASSETS = @path joinpath(@__DIR__, "..", "assets")
```

## Native desktop installers: `AppBundler.jl`

`create_app` produces a raw relocatable folder; end-users on macOS, Windows, and Linux expect
native installers, icons, desktop entries, sandboxing, and code signing.
[`AppBundler.jl`](https://github.com/PeaceFounder/AppBundler.jl) bridges this gap by packaging
Julia GUI and desktop applications into:

- **macOS**: `.app` application bundles and signed `.dmg` disk images
- **Windows**: `MSIX` installers
- **Linux**: `Snap` packages

It manages staging, artifact caching, code-signing hooks, and evolving support for trimmed
Julia 1.12+ builds to keep GUI installer sizes down.

## `PackageCompiler.jl` vs. `JuliaC.jl`

- Choose **`PackageCompiler.jl`** for local developer sysimages, complex GUI or data-science
  apps that depend on dynamic dispatch / `DataFrames` / arbitrary packages, or mature
  pre-1.12 LTS deployments.
- Choose **[`JuliaC.jl`](/compilers/juliac/)** when you are on Julia 1.12+ and want small
  trimmed binaries (`--trim=safe`), `--export-abi` + [`JuliaLibWrapping.jl`](/bindings/julialibwrapping/),
  or `--privatize` for loading multiple Julia libraries in one host process.
