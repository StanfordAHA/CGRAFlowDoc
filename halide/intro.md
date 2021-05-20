# Introduction of Halide-to-CoreIR
## Overview of Halide

Halide is a programming language designed to make it easier to write
high-performance image and array processing code on modern machines.
The language is embedded in C++ with a functional language built on top
to allow definition of variables based on x and y coordinates. Writing
Halide is unique in that the output values are captured in the _algorithm_, 
while the execution speed is defined by a target-specific _schedule_. Read
[Writing Applications](writing-apps.md) for more information on writing 
Halide code.

The following function defines and sets the schedule for a 3x3 box filter
defined as a series of two 3x1 passes:

```C++
Func blur_3x3(Func input) {
  Func blur_x, blur_y;
  Var x, y, xi, yi;

  // The algorithm - no storage or order
  blur_x(x, y) = (input(x-1, y) + input(x, y) + input(x+1, y))/3;
  blur_y(x, y) = (blur_x(x, y-1) + blur_x(x, y) + blur_x(x, y+1))/3;

  // The schedule - defines order, locality; implies storage
  blur_y.tile(x, y, xi, yi, 256, 32)
        .vectorize(xi, 8).parallel(y);
  blur_x.compute_at(blur_y, x).vectorize(x, 8);

  return blur_y;
}
```

## Target Architecture: CoreIR and Clockwork

Note that the above box filter schedules to hardware that has a vector unit
of size 8. This value may not be compatible based on the hardware target.
Main-line Halide currently targets:

* CPU architectures: X86, ARM, MIPS, Hexagon, PowerPC
* Operating systems: Linux, Windows, macOS, Android, iOS, Qualcomm QuRT
* GPU Compute APIs: CUDA, OpenCL, OpenGL, OpenGL Compute Shaders, Apple Metal,
  Microsoft Direct X 12

This project extends the available targets to custom-hardware generation. The language
that is used capture hardware intent is CoreIR. CoreIR's most basic features provide a
way to create hardware modules (adders, multipliers, muxes) and connections (how to 
wire them together). Beyond this, there are analysis and transformation passes to modify 
the circuit. More information can be found in the [CoreIR section](coreir/intro.md). Instead
of directly creating CoreIR, the memories are compiled and analyzed by Clockwork.

## Input and Output
The input is a Halide application file (with algorithm and schedule) including a target
to Clockwork. After compilation, a pair of files are created: a Clockwork memory file, and
a CoreIR json file containing the circuits for the computation  kernels of the application.
In addition, each design also compiles a CPU version of the same application. This CPU version
is used to take an input image, and produce a golden reference output image. More information
on usage, as well as which files are generated, can be found in [Usage Instructions](usage.md).


## Getting Started

* Repo:
  https://github.com/StanfordAHA/Halide-to-Hardware/

You should be able to compile by simply invoking `make distrib`
in the top level folder. Once the Halide compiler is created, a `make all` command
in the application folder will create the CoreIR files.

```sh
cd Halide-to-Hardware/apps/hardware_benchmarks/tests/conv_3_3
make all
```
