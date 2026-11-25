+++
title = "Compiling Julia to WebAssembly in 2026: WasmTarget.jl, WasmCodegen.jl, and Snapshot.jl"
date = 2026-07-15
[params]
author = "JuliaCompilers Contributors"
+++

Browser-native WasmGC and closed-world type inference enabled a new generation of Julia-to-WebAssembly
compilers in 2026, powering standalone `.wasm` modules, `Therapy.jl` web islands, and
zero-server interactive Pluto notebooks via `Snapshot.jl`.

<!--more-->

Two complementary approaches have driven Julia's 2026 WebAssembly momentum:

1. **[`WasmTarget.jl`](/compilers/webassembly/)**: Registered in April 2026 alongside the
   `Therapy.jl` islands web framework and `Snapshot.jl` static Pluto exporter, compiling
   interactive Julia UI islands and notebook cells directly into compact WebAssembly modules.
2. **[`WasmCodegen.jl`](/compilers/webassembly/)** (building on ideas from
   `WebAssemblyCompiler.jl` and `juliac --trim`): Performs closed-world reachability analysis
   from entry points, serializes required Julia objects, and lowers inferred IR directly to
   **WasmGC** bytecode using the browser's built-in garbage collector—requiring zero `libjulia`.

Combined with [`WebAssemblyInterfaces.jl`](https://github.com/tshort/WebAssemblyInterfaces.jl)
for JavaScript type generation, pure-Julia scientific solvers and interactive teaching
notebooks now run client-side in Chrome, Firefox, and Safari with zero server infrastructure.

See the [WebAssembly & Browser Targets guide](/compilers/webassembly/) for details and current
constraints.
