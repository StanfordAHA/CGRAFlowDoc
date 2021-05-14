# Hardware Lowering
There are several small lowering passes that are unique to hardware for generating
Clockwork. These passes extract the hardware accelerators, retain n-d memory indexing,
merge unrolled memories, inline memory constants, and retain BFloat math.

## Hardware Accelerator Extraction
This pass identifies which functions have been accelerated with the `hw_accelerate()` scheduling
call. The store loop is saved, and then the HalideIR is modified to denote where the accelerator
should be produced. A produce node creating `_hls_target.` is created inside the store loop.
Later, the codegen uses this node to know when to start generating Clockwork code.

## Multi-dimensional Memory Indexing
Multi-dimensional loads (calls) and stores (provides) are typically flattened to one-dimensional
memories. However, the multi-dimensional indexing can be helpful for Clockwork. Therefore,
the convention we use is that by appending `.stencil` to a function, the memory is not flattened,
and is recognized as function used in a hardware accelerator.

## Unrolled Memory Update Merging
When a reduction loop is unrolled, typically the updates all occur as different memory
operations. However, this would produce many updates and small computation kernels for
Clockwork. Instead, we combine iterative updates from unrolled reduction loops into a single
computation.

## Memory Constant Inlining
Some functions in Halide are created that look like memories, but in Clockwork are easier to
treat as computation blocks. This is when they are preloaded with values and then only stores
are performed. They can either be `ROM`s or `CONSTS` based on the nature of the loads and stores.
When this occurs, they are transformed into computation by removing the `realization` node.

## BFloat Math
The CGRA has BFloat operators, so those are native operations. However, a CPU natively can only
perform 32 bit floating point operations, so emulation must be done to get bit-exact comparisons
on the CPU and CGRA outputs. `EmulateFloat16Math.cpp` performs these math conversions to
ensure the output of each hardware target is consistent. Note that during emulation, each CPU
BFloat operation instead takes many operations to execute (including casts and shifts).

