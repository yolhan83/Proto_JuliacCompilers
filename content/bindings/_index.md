+++
title = "Bindings & Add-ons"
description = "Automatically generate C headers, Python packages, Rust crates, JavaScript/Wasm interfaces, FMUs, and bi-directional language bridges."
+++

Compiling Julia code is only half the story—most real deployments need other languages to
**call** compiled Julia libraries (or need Julia to wrap massive C/C++/Python ecosystems).
In 2025–2026, automatic binding tooling split cleanly into three categories:

1. **AOT self-contained exports (no Julia install required on target)**: Powered by
   [`JuliaC.jl`](/compilers/juliac/) (`--export-abi`, `--privatize`),
   [`JuliaLibWrapping.jl`](/bindings/julialibwrapping/),
   [`PackageCompiler.jl`](/compilers/packagecompiler/) (`create_library`), and
   [`StaticCompiler.jl`](/compilers/static-compiler/) (`compile_shlib`).
2. **Live-runtime bridges**: Embed or connect to a full dynamic Julia runtime from Python
   (`JuliaCall` / `PythonCall.jl`), Rust (`jlrs`), R (`JuliaConnectoR`), or MATLAB.
3. **Foreign-to-Julia automatic binding generators**: Parse C/C++ headers with LLVM/Clang to
   auto-generate Julia wrappers (`Clang.jl`, `CBinding.jl`, `CxxWrap.jl` + `WrapIt`).

| Target language | Self-contained AOT export (no Julia install) | Live-runtime / bi-directional bridge |
|-----------------|----------------------------------------------|--------------------------------------|
| **Python** | [`JuliaLibWrapping.jl`](/bindings/julialibwrapping/) (`ctypes` + NumPy) · `JuliaC` wheels | [`PythonCall.jl` / `JuliaCall`](/bindings/python-r-web/#python-pythoncalljl--juliacall--compiled-wheels) |
| **C / Fortran** | `JuliaC --output-lib --export-abi` · `JuliaLibWrapping.jl` · `StaticCompiler.compile_shlib` · `PackageCompiler.create_library` · `Symbolics.CTarget()` | [`Clang.jl` & `CBinding.jl`](/bindings/c-cpp-rust/#auto-generating-julia-wrappers-for-c-clangjl--cbindingjl) |
| **C++** | `JuliaC --output-lib --privatize` (C ABI) · `PackageCompiler.create_library` | [`CxxWrap.jl` + `WrapIt`](/bindings/c-cpp-rust/#auto-generating-julia-wrappers-for-c-cxxwrapjl--wrapit) |
| **Rust** | Link `JuliaC` / `StaticCompiler` shared lib via `extern "C"` | [`jlrs`](/bindings/c-cpp-rust/#rust--julia-jlrs) (embed Julia or export Rust via `julia_module!`) |
| **R** | Load `JuliaC` shared lib via `.C()` / `.Call()` · JLW R backend *(in progress)* | [`JuliaConnectoR` & `RCall.jl`](/bindings/python-r-web/#r-juliaconnector--rcalljl) |
| **MATLAB / Simulink** | `loadlibrary` / MEX over `JuliaC` · `FMIExport.jl` (`.fmu`) · `Symbolics.MATLABTarget()` · JLW MATLAB *(in progress)* | [`MATLAB.jl`](/bindings/python-r-web/#matlab-excel--java) |
| **JavaScript / Browser** | [`WasmTarget.jl` / `WasmCodegen.jl` + `WebAssemblyInterfaces.jl`](/compilers/webassembly/) | [`Therapy.jl` & `Snapshot.jl`](/compilers/webassembly/) |
| **Microservices / ML** | [`ReactantServer.jl`](/compilers/accelerators-mlir/) (StableHLO + KServe V2 gRPC) | HTTP / gRPC servers |
