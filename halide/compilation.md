# Compilation
With a completed Halide algorithm and schedule, it must now be passed through
the compiler to generate CoreIR. This modified backend includes all passes
from the original compiler, and it adds a couple passes specifically for
hardware generation. The full set of passes can be found in `src/Lower.cpp`.

The hardware passes added are Linebuffer Extraction to find where and what
size the linebuffers should be inserted into HalideIR, and Stream Optimization
to change execution from sequential to simultaneous execution of stages.
These are described in more detail in [Hardware Lowering](hardware-lowering.md).

## HalideIR: Loop Nests
Halide frontend language is a mix of functional definitions in the algorithm
and imperative scheduling functions. However, the backend compilation is 
represented in a different representation for ease of compiling. This 
internal representation, HalideIR, is an abtract syntax tree of the operations
for the Halide program. It contains the operators as well as control and 
memory elements.

The first step of lowering, `schedule_functions` transforms the frontend
language to a set of loop nests that define the program execution. This
can be visualized by outputting the `Stmt` before and after each compilation
stage as seen in the debug statements in `Lower.cpp`. An example is shown
below:
```C++
std::cout << "Lowering before sliding window:\n" << s << '\n';
std::cout << "Performing sliding window optimization...\n";
s = sliding_window(s, env);
std::cout << "Lowering after sliding window:\n" << s << '\n';
```
