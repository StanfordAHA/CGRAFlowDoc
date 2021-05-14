# Code Generation
Following the lowering passes, the HalideIR must be translated into Clockwork.
The Clockwork codegen extends the functions for generating C code. The codegen
produces not only the Clockwork memory inputs, but also the CoreIR json for the
computation kernels, and a surrounding test harness. The test harness is the CPU code that
is needed to launch the CoreIR design to fully execute the image.

## Clockwork codegen
The basic working of the codegen is that the abstract syntax tree (AST) is
traversed and the circuit is built up from input to output. Each operator
in HalideIR have defined actions using `visit` functions, similar to LLVM.

The main function that creates each accelerator is generated when a `_hls_target` is
found in the AST. These nodes are created based on the `hw_accelerate` scheduling
calls in the Halide schedule. This triggers `add_kernel()`, which in turn creates
all necessary files for the accelerator.

The memory stores (Provide in the Halide AST) are the core nodes that generate the
Clockwork code. Each memory store is associated with zero or more memory loads. To
connect the store with the loads, a unique computation kernel is created in another file.

## CoreIR codegen
The CoreIR codegen (`CoreIRCompute.cpp`) is called for each computation kernel.
The Halide AST is processed to generate all of the operators and connections.

Similar to the C codegen, the variables that have been defined are stored
in a data structure, and subsequent operators recall these variables. In a
hardware sense, the wire names are remembered, and new modules use previously
defined hardware to connect to their inputs, and define a new variable that
is the output.

### Basic Operators
Most basic arithmetic operators (add, multiply, xor, select, abs) follow
the same codegen procedure.
For example, `a = b + c` would be seen in HalideIR as a call to `visit(Add *op)`.
The CoreIR codegen would find the wires defined by `b` and `c`. Then, in 
CoreIR a new adder would be created into our CoreIR design. Last, the `b` and
`c` wires would be connected to the adder inputs, and the adder output
would be stored as `c`. Most arithmetic operations are quite simple, and
are treated very similar to the above example.

### Design Input and Output
The design input and output are defined in the beginning of the design
creation. These special variables are remembered, since they are wires
that are connected to the input/output ports instead of module ports.

The input and output ports are iteratively stored in a vector before the
CoreIR module is created. This way, we are able to flexibly create the
ports for the entire module regardless of how many inputs and outputs,
and the datatypes of each.

### Constant
When any wire is encountered, first the operator is analyzed to determine
if it is a constant. Since we do not want to reuse constants, a new
constant is generated each time and stored in a new constant register.

### Load from an array
A load from an array necessitates different memory structure depending
on how it is performed. These two properties are the indexing (constant/variable)
and values (constant/variable). Consider,
```C++
int read_value = array[index];
```

If `index` is always the same value, then whatever wire defines that 
array element can be used.

If `index` is a variable, then the output value changes during program 
execution. In hardware, we need some indexing logic or a mux. If the values
of the array are constants, then we have a read-only memory (ROM). 
The ROM is created and initialized with values, and indexing logic is
constructed based on the calculation of `index`.

If the array is indexed with a variable, and values of the array are not constant, then a mux must
be created. Further passes are used to find the possible values of `index`
and wire these array elements to an appropriately sized mux. Note that `index`
is never instantiated in hardware (unlike the ROM), and instead `index`
manifests in the hardware as connections to different mux inputs.

## Test harness codegen
This additional file provides the execution for the CPU to launch and feed
data into the accelerator. This creates the input image (usually with the
padding that is not expected to be done on the accelerator) and feeds
the input image one tile at a time (since the input image is likely tiled
to reduce the linebuffer sizes). This file is named `clockwork_testscript.cpp`.
