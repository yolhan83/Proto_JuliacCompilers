+++
title = "MLIR/XLA, GPU & AD Compilers"
description = "Whole-program MLIR/StableHLO/XLA compilation across CPUs, GPUs, and TPUs, LLVM-level automatic differentiation, tile-based Tensor Core IR, and reusable GPU compiler infrastructure."
weight = 6
badge = "Accelerators"
badgeNew = true

[[params.packages]]
name = "Reactant.jl"
repository = "https://github.com/EnzymeAD/Reactant.jl"
documentation = "https://enzymead.github.io/Reactant.jl/stable/"

[[params.packages]]
name = "Enzyme.jl"
repository = "https://github.com/EnzymeAD/Enzyme.jl"
documentation = "https://enzymead.github.io/Enzyme.jl/stable/"

[[params.packages]]
name = "GPUCompiler.jl"
repository = "https://github.com/JuliaGPU/GPUCompiler.jl"
documentation = "https://juliagpu.github.io/GPUCompiler.jl/stable/"

[[params.packages]]
name = "cuTile.jl"
repository = "https://github.com/JuliaGPU/cuTile.jl"
documentation = "https://juliagpu.github.io/cuTile.jl/dev/"

[[params.packages]]
name = "LLVM.jl"
repository = "https://github.com/maleadt/LLVM.jl"
documentation = "https://maleadt.github.io/LLVM.jl/stable/"
+++

Beyond CPU executables and shared libraries, Julia hosts some of the most sophisticated
**domain-specific and hardware-accelerator compilers** in scientific computing—all driven by
Julia's ability to intercept and transform typed IR, LLVM IR, and MLIR.

## Whole-program MLIR & XLA compilation: `Reactant.jl`

[`Reactant.jl`](https://github.com/EnzymeAD/Reactant.jl) traces generic Julia code (using
`ConcreteRArray` / `TracedRArray`), lowers it to **MLIR / StableHLO**, and compiles the entire
program through Google's **XLA** compiler stack.

Key capabilities in 2025–2026:
- **Multi-accelerator targets**: Compiles one Julia codebase to **CPUs** (`x86_64` / `aarch64`),
  **NVIDIA GPUs** (CUDA/NVVM), **AMD GPUs** (ROCm), **Google Cloud TPUs** (via PJRT), and
  experimental accelerators (such as Tenstorrent).
- **Aggressive whole-program optimization**: Eliminates Julia type instabilities and dynamic
  dispatch inside traced regions, fuses elementwise and reduction loops, and removes temporary
  allocations.
- **MLIR-level automatic differentiation (`EnzymeMLIR`)**: Integrates directly with Enzyme at
  the MLIR level for high-performance forward- and reverse-mode AD, pairing seamlessly with
  deep learning frameworks like [`Lux.jl`](https://lux.csail.mit.edu/).
- **AOT compilation & production serving**: Use `@compile` to compile ahead of time, export
  StableHLO model bundles (`ReactantNitro` / `ReactantServerExport`), and deploy them in
  production without Python via [`ReactantServer.jl`](https://github.com/EnzymeAD/Reactant.jl)
  using standard **KServe V2** gRPC (or Triton Inference Server integration).

```julia
using Reactant

function loss(W, b, x, y)
    pred = W * x .+ b
    return sum(abs2, pred .- y)
end

W = Reactant.to_rarray(randn(Float32, 64, 128))
b = Reactant.to_rarray(zeros(Float32, 64))
x = Reactant.to_rarray(randn(Float32, 128, 32))
y = Reactant.to_rarray(randn(Float32, 64, 32))

# Compile function + reverse-mode gradient ahead of time via MLIR/XLA + EnzymeMLIR
grad_fn = @compile Enzyme.gradient(Reverse, loss, W, Const(b), Const(x), Const(y))
dW, = grad_fn(W, b, x, y)
```

## LLVM-level automatic differentiation: `Enzyme.jl`

[`Enzyme.jl`](https://github.com/EnzymeAD/Enzyme.jl) performs automatic differentiation
directly on optimized **LLVM IR** rather than high-level source expressions. Because it acts
after Julia's type inference and LLVM's optimization passes, it differentiates mutation,
in-place loops, scalar code, GPU kernels, and even foreign C/C++ routines with near-zero
runtime overhead.

## Reusable LLVM & GPU compiler stack

For hardware-specific kernel compilation, Julia's GPU ecosystem—covered in depth at
[juliagpu.org](https://juliagpu.org/)—is built on reusable compiler packages:

- **[`LLVM.jl`](https://github.com/maleadt/LLVM.jl)**: Full Julia bindings to LLVM's IR
  builder, pass managers, target machines, and ORC JIT.
- **[`GPUCompiler.jl`](https://github.com/JuliaGPU/GPUCompiler.jl)**: Reusable compiler driver
  that hooks into Julia's type inference and emits target-specific LLVM/machine code. It powers
  `CUDA.jl` (PTX), `AMDGPU.jl` (AMDGCN), `Metal.jl` (Apple AIR), `oneAPI.jl` (SPIR-V),
  `OpenCL.jl`, as well as [`StaticCompiler.jl`](/compilers/static-compiler/) and
  [`WasmTarget.jl`](/compilers/webassembly/).
- **[`cuTile.jl`](https://github.com/JuliaGPU/cuTile.jl)** *(2026)*: Compiles tile-based Julia
  kernels down to **NVIDIA Tile IR**, targeting Tensor Cores and Tensor Memory Accelerators
  from high-level multi-dimensional array fragments instead of manual thread indexing.
- **[`KernelAbstractions.jl`](https://github.com/JuliaGPU/KernelAbstractions.jl) &
  [`AcceleratedKernels.jl`](https://github.com/JuliaGPU/AcceleratedKernels.jl)**: Portable
  `@kernel` programming and data-parallel algorithms (sorting, scans, reductions) compiled
  across CUDA, ROCm, Metal, oneAPI, OpenCL, and multi-threaded CPUs.
- **[`SPIRV.jl`](https://github.com/serenity4/SPIRV.jl)**: Pure-Julia toolkit for constructing,
  validating, transforming, and emitting portable SPIR-V shader/compute binaries (Vulkan and
  OpenCL).
