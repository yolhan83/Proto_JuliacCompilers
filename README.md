# JuliaCompilers website

A community hub for everything related to compilation and foreign-language bindings in Julia—modeled
directly on the architecture and design of [JuliaGPU/juliagpu.org](https://github.com/JuliaGPU/juliagpu.org).
The site is built with [Hugo](https://gohugo.io/).

## What is covered

- **Compilers (`/compilers/`)**:
  - [REPL, JIT & Precompilation](content/compilers/jit-repl.md): `Core.Compiler`, `pkgimages`,
    `PrecompileTools.jl`, `SnoopCompile.jl`, `Revise.jl`, `DaemonMode.jl`, `Pkg.Apps`,
    `BaseCompiler.jl` (`Compiler` stdlib), `JuliaSyntax.jl`, `JuliaLowering.jl`, `JET.jl`,
    `JETLS.jl`, `Cthulhu.jl`, `DispatchDoctor.jl`.
  - [JuliaC.jl & Trimming](content/compilers/juliac.md): `--output-exe`, `--output-lib`,
    `--output-sysimage`, `--output-o`, `--output-bc`, `--trim=safe/unsafe-warn/unsafe/no`,
    `--bundle`, `--export-abi`, `--privatize`, `ImageRecipe` / `LinkRecipe` / `BundleRecipe`.
  - [PackageCompiler.jl & Desktop Bundling](content/compilers/packagecompiler.md):
    `create_sysimage`, `create_app`, `create_library`, `AppBundler.jl` (DMG/MSIX/Snap),
    `RelocatableFolders.jl`.
  - [StaticCompiler.jl & No-GC Compilation](content/compilers/static-compiler.md):
    `StaticCompiler.jl`, `StaticTools.jl`, `StaticRuntime.jl`, `Bumper.jl`, `AllocArrays.jl`,
    `AllocCheck.jl`.
  - [WebAssembly & Browser Targets](content/compilers/webassembly.md):
    `WasmTarget.jl`, `WasmCodegen.jl`, `WebAssemblyCompiler.jl`, `WebAssemblyInterfaces.jl`,
    `Therapy.jl`, `Snapshot.jl`.
  - [MLIR/XLA, GPU & AD Compilers](content/compilers/accelerators-mlir.md):
    `Reactant.jl`, `ReactantServer.jl`, `Enzyme.jl`, `GPUCompiler.jl`, `LLVM.jl`, `cuTile.jl`,
    `KernelAbstractions.jl`, `AcceleratedKernels.jl`, `SPIRV.jl`.
  - [Symbolic Codegen, FMUs & Extensibility](content/compilers/symbolic-fmi.md):
    `Symbolics.jl` (`CTarget`, `StanTarget`, `MATLABTarget`), `ModelingToolkit.jl`,
    `FMIExport.jl`, `FMIBuild.jl`, `CassetteOverlay.jl`, `IRTools.jl`.
- **Bindings & Add-ons (`/bindings/`)**:
  - [JuliaLibWrapping.jl (Python & C)](content/bindings/julialibwrapping.md): `JuliaLibWrapping.jl`
    & `JLWInterop` (`@api`, auto C headers & pip-installable Python `ctypes`/NumPy packages).
  - [C, C++, Fortran & Rust](content/bindings/c-cpp-rust.md): `jlrs`, `CxxWrap.jl`, `WrapIt`,
    `Clang.jl`, `CBinding.jl`, `BinaryBuilder.jl`, plus AOT shared-library exports.
  - [Python, R, MATLAB, Java & Web](content/bindings/python-r-web.md): `PythonCall.jl` /
    `JuliaCall`, `JuliaConnectoR`, `RCall.jl`, `MATLAB.jl`, `JavaCall.jl`,
    `WebAssemblyInterfaces.jl`, `FMIExport.jl`, `ReactantServer.jl`.
- **Guides & Reference**:
  - [Get started](content/get-started.md) with synchronized pure-CSS workflow tabs
  - [Compare compilers](content/compare.md) matrix & benchmarks
  - [Ecosystem directory](content/ecosystem.md) (45+ packages)
  - [Showcases](content/showcases.md) & [Blog](content/post/_index.md)

## Building & previewing locally

Install [Hugo](https://gohugo.io/installation/) (v0.158.0+, CI uses the version pinned in
`.github/actions/setup-hugo/action.yml`, e.g. `brew install hugo` on macOS), then run:

```sh
hugo server
```

Open <http://localhost:1313/> in your browser.

To run the exact strict production build used in GitHub Actions:

```sh
hugo build --minify --panicOnWarning
```

## Writing a blog post

```sh
hugo new content post/2026-12-01-my-announcement.md
```

The first paragraph up to `<!--more-->` is used as the summary on the home page and RSS feed.
REPL and CLI sessions go in `julia-repl` code blocks.
