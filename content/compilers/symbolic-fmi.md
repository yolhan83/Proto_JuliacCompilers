+++
title = "Symbolic Codegen, FMUs & Extensibility"
description = "Compile symbolic math to standalone C/Stan/MATLAB source, export Functional Mock-up Units (FMUs) accelerated by JuliaC, and write custom Julia compiler passes."
weight = 7
badge = "Domain & IR"

[[params.packages]]
name = "Symbolics.jl"
repository = "https://github.com/JuliaSymbolics/Symbolics.jl"
documentation = "https://symbolics.juliasymbolics.org/stable/"

[[params.packages]]
name = "FMIExport.jl"
repository = "https://github.com/ThummeTo/FMIExport.jl"
documentation = "https://thummeto.github.io/FMI.jl/stable/"

[[params.packages]]
name = "FMIBuild.jl"
repository = "https://github.com/ThummeTo/FMIBuild.jl"
documentation = "https://thummeto.github.io/FMI.jl/stable/"

[[params.packages]]
name = "CassetteOverlay.jl"
repository = "https://github.com/JuliaDebug/CassetteOverlay.jl"
documentation = "https://github.com/JuliaDebug/CassetteOverlay.jl#readme"
+++

Not every compilation problem targets `libjulia` or LLVM directly. Julia also excels at
**symbolic-to-source code generation**, **industry-standard simulation packaging (FMUs)**, and
**user-defined compiler passes** that transform Julia IR before LLVM ever sees it.

## Symbolic compilation to C, Stan, MATLAB & Julia: `Symbolics.jl`

[`Symbolics.jl`](https://github.com/JuliaSymbolics/Symbolics.jl) (and
[`ModelingToolkit.jl`](https://github.com/SciML/ModelingToolkit.jl)) can trace numerical Julia
functions into symbolic graphs, simplify and differentiate them symbolically, and compile them
via `build_function` into fast standalone source code for multiple target languages:

- `Symbolics.JuliaTarget()` — optimized Julia functions (optionally compiled via `RuntimeGeneratedFunctions`)
- `Symbolics.CTarget()` — pure, dependency-free **C source code** that can be compiled with
  `gcc`/`clang` (zero Julia JIT latency even on huge generated Jacobians!)
- `Symbolics.StanTarget()` — Stan probabilistic programming code
- `Symbolics.MATLABTarget()` — MATLAB function source

```julia
using Symbolics

@variables x y
expr = [sin(x) * y^2, cos(y) + x * y]

# Generate standalone C code for the expression and its symbolic Jacobian
J = Symbolics.jacobian(expr, [x, y])
c_code = build_function(J, [x, y]; target = Symbolics.CTarget())
```

## Functional Mock-up Units (FMUs): `FMIExport.jl` & `FMIBuild.jl`

In automotive, aerospace, robotics, and Model-Based Engineering (MBE), models are exchanged
between tools (Simulink, Dymola, ANSYS, OpenModelica) as standard **Functional Mock-up Units
(`.fmu`)**.

- **[`FMIExport.jl`](https://github.com/ThummeTo/FMIExport.jl)**: Exports arbitrary Julia
  differential-equation and control models as FMI-compliant Co-Simulation and Model-Exchange
  FMUs.
- **[`FMIBuild.jl`](https://github.com/ThummeTo/FMIBuild.jl)**: Separates build/bundling
  dependencies from the runtime export package so exported models remain lean.
- **2025–2026 `JuliaC` acceleration**: Historically, exporting a Julia FMU relied on
  `PackageCompiler.jl` sysimages (hundreds of megabytes and long build times). Newer workflows
  (including JuliaHub's **Dyad** platform) leverage `JuliaC.jl` and trimming, slashing FMU
  export build times by up to ~10× and shrinking FMU archives down to ~58–60 MB.

## Custom compiler passes & contextual dispatch

Because Julia's compiler is written largely in Julia itself, package authors can extend it
without forking the language:

- **[`CassetteOverlay.jl`](https://github.com/JuliaDebug/CassetteOverlay.jl)**: The modern,
  lightweight successor to `Cassette.jl`. Built on Julia's native
  `Base.Experimental.@overlay` and `Core.Compiler.OverlayMethodTable`, it enables contextual
  dispatch (intercepting calls within a dynamic scope, e.g. for automatic differentiation,
  tracing, or hardware overrides) without the fragility of legacy IR overdubbing.
- **[`BaseCompiler.jl` (`Compiler` stdlib)](https://github.com/JuliaLang/BaseCompiler.jl)**:
  Exposes Julia's compiler as a Pkg-switchable standard library (Julia 1.12+), letting compiler
  researchers and tool authors test custom `AbstractInterpreter` implementations or compiler
  branches per project.
- **[`JuliaLowering.jl`](https://github.com/JuliaLang/JuliaLowering.jl) &
  [`JuliaSyntax.jl`](https://github.com/JuliaLang/JuliaSyntax.jl)**: Pure-Julia syntax parsing
  and lowering front-end with exact source-location provenance.
- **[`IRTools.jl`](https://github.com/FluxML/IRTools.jl)**: High-level utilities for inspecting
  and transforming Julia SSA IR (historically powering source-to-source AD and compiler
  experiments).
