+++
title = "WebAssembly & Browser Targets"
description = "Compile pure Julia directly to WebAssembly and WasmGC—powering client-side scientific solvers, zero-server Pluto notebooks, and interactive web islands."
weight = 5
badge = "2025–2026"
badgeNew = true

[[params.packages]]
name = "WasmTarget.jl"
repository = "https://github.com/GroupTherapyOrg/WasmTarget.jl"
documentation = "https://github.com/GroupTherapyOrg/WasmTarget.jl#readme"

[[params.packages]]
name = "WasmCodegen.jl"
repository = "https://github.com/gbaraldi/WasmCodegen.jl"
documentation = "https://github.com/gbaraldi/WasmCodegen.jl#readme"

[[params.packages]]
name = "WebAssemblyCompiler.jl"
repository = "https://github.com/tshort/WebAssemblyCompiler.jl"
documentation = "https://tshort.github.io/WebAssemblyCompiler.jl/dev/"

[[params.packages]]
name = "WebAssemblyInterfaces.jl"
repository = "https://github.com/tshort/WebAssemblyInterfaces.jl"
documentation = "https://github.com/tshort/WebAssemblyInterfaces.jl#readme"
+++

Running Julia natively in web browsers without shipping a 50+ MB Emscripten build of `libjulia`
was a long-standing challenge because of Julia's garbage collector and asynchronous task
runtime. In **2025–2026**, widespread browser support for **WebAssembly Garbage Collection
(WasmGC)** and closed-world type inference unlocked a wave of lightweight Julia-to-WebAssembly
compilers.

## Two compilation strategies

### 1. Closed-world WasmGC codegen (`WasmCodegen.jl`, `WebAssemblyCompiler.jl`)

Rather than porting `libjulia` to WebAssembly, WasmGC compilers read Julia's fully type-inferred
IR (`Base.code_typed`) and emit WebAssembly GC structs, arrays, and functions directly:

- **[`WasmCodegen.jl`](https://github.com/gbaraldi/WasmCodegen.jl)**: Inspired by
  `juliac --trim`, it traces a closed world of reachable methods from declared entry points,
  serializes required Julia objects straight into the Wasm module, and provides a minimal
  pure-Julia runtime (math, strings, basic I/O) running on the browser's own garbage collector
  with zero `libjulia` dependency. It targets modern Julia (1.12/1.13) and can compile math
  solvers, data structures, and even parts of `JuliaSyntax` / `OrdinaryDiffEq`.
- **[`WebAssemblyCompiler.jl`](https://github.com/tshort/WebAssemblyCompiler.jl)**: The
  foundational exploratory Julia-to-WasmGC compiler from `tshort`, pioneering direct IR lowering
  to WasmGC bytecode and typed browser arrays.

### 2. `GPUCompiler`-based & islands targets (`WasmTarget.jl`)

- **[`WasmTarget.jl`](https://github.com/GroupTherapyOrg/WasmTarget.jl)** *(registered April
  2026)*: Compiles type-inferred Julia code to WebAssembly (leveraging `GPUCompiler.jl` /
  `wasm32-unknown-wasi` and Wasm bytecode emission) so self-contained interactive Julia
  components run directly in browser runtimes or WASI hosts.

## Full-stack frameworks & zero-server notebooks (2026)

Built directly on top of `WasmTarget.jl`, two 2026 packages bring Julia compilation to web
publishing and interactive notebooks:

- **[`Therapy.jl`](https://github.com/GroupTherapyOrg/Therapy.jl)**: A document-first
  "islands architecture" web framework written in Julia. Pages render HTML on the server, while
  interactive components marked `@island` compile down to compact WebAssembly with fine-grained
  reactive signals—no heavy JavaScript framework required.
- **[`Snapshot.jl`](https://snapshot.show/)**: Exports interactive
  [Pluto.jl](https://plutojl.org/) notebooks into static HTML + WebAssembly bundles. Sliders,
  buttons, and numerical cells execute directly in the reader's browser with **zero live Julia
  server** required.

## Bridging JavaScript & Julia Wasm: `WebAssemblyInterfaces.jl`

When calling compiled `.wasm` modules from custom JavaScript or TypeScript frontends,
[`WebAssemblyInterfaces.jl`](https://github.com/tshort/WebAssemblyInterfaces.jl) inspects
Julia structs and types to generate matching JavaScript type definitions and shared-memory
bindings (compatible with `wasm-ffi`), removing manual pointer offset boilerplate.

## Current status & trade-offs

WebAssembly compilation in Julia is **rapidly advancing but experimental**:
- **Strengths**: Tiny browser-native `.wasm` artifacts, instant client-side reactivity, zero
  server hosting costs, and strong alignment with browser WasmGC engines (Chrome, Firefox,
  Safari).
- **Limitations**: Closed-world/type-stable subset required; packages that call native C/Fortran
  JLL libraries (`ccall`) cannot run directly in browser Wasm unless replaced by pure-Julia
  implementations.
