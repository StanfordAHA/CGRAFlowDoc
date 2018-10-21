# Code Generation
Following the lowering passes, the HalideIR must be translated into CoreIR.
The CoreIR codegen extends the functions for generating C code. The codegen
produces not only the CoreIR json design, but also creates a debugging
file and a surrounding test harness. The debugging file is structured as
a C++ file with additional comments. The test harness is the CPU code that
is needed to launch the CoreIR design to fully execute the image.

## CoreIR codegen
The basic working of the codegen is that the abstract syntax tree (AST) is
traversed and the circuit is built up from input to output. Each operator
in HalideIR have defined actions using `visit` functions, similar to LLVM.

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
and the datatypes of each. Furthermore, based on the target interface, we can
also conditionally add a reset or valid port to the design.

### Linebuffer
The linebuffer is defined by parameters extracted during lowering. These
parameters match with the CoreIR linebuffer parameters. This library
generates the necessary CoreIR hardware from the input linebuffer parameters.
Therefore, the Halide compiler is not responsible for doing the wiring
or instantiation of the registers or rowbuffers for this stage. 

The linebuffer is thus instantiated in the codegen with the inputs connected
and the output stencil added based on the linebuffer name.

When valid signals are being used, linebuffers are also connected to each 
other with a linebuffer output valid connected to subsequent linebuffer's
write enable.

### Counter
When a `for` loop is encountered, a counter may be needed in the generated
CoreIR hardware. However, some of the counter are merely the iteration
loops on the `x` and `y` variables. The loop body is analyzed and only
if the loop variable is used in the loop body is a counter created.

### Constant
When any wire is encountered, first the operator is analyzed to determine
if it is a constant. Since we do not want to reuse constants, a new
constant is generated each time and stored in a new constant register.

### Load from an array
A load from an array necessitates different memory structure depending
on how it is performed. These two properties are the indexing (constant/variable)
and values (constant/variable). Consider,
```
int read_value = array[index];
```

If `index` is always the same value, then whatever wire defines that 
array element can be used.

If `index` is a variable, then the output value changes during program 
execution. In hardware, we need some indexing logic or a mux. If the values
of the array are constants, then we have a read-only memory (ROM). 
The ROM is created and initialized with values, and indexing logic is
constructed based on the calculation of `index`.
For ROMs of a small size (fewer than 16 elements), we decide to use registers
and muxes instead as described below. 

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
to reduce the linebuffer sizes).