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

## Target Architecture: CoreIR

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
the circuit. More information can be found in the [CoreIR section](coreir/intro.md).

## Input and Output
The input is a Halide application file (with algorithm and schedule) including a target
to CoreIR. After compilation, a CoreIR json file contains the circuit for the application.
In addition, each design also compiles a CPU version of the same application. This CPU version
is used to take an input image, and produce a golden reference output image. More information
on usage, as well as which files are generated, can be found in [Usage Instructions](usage.md).


## Getting Started
There are two Halide repo that Jeff is maintaining. Although one of them is
obsolete, it has most of the applications that the CGRAFlow is testing. We
expect the old repo is phasing out soon.

* Old repo:
  https://github.com/jeffsetter/Halide_CoreIR
* New repo:
  https://github.com/StanfordAHA/Halide-to-Hardware/

You should be able to compile both of them simply by invoking `make` command
in the top level folder. Once the Halide compiler is created, a `make all` command
in the coreir application folder will create the CoreIR files.

```
cd Halide_CoreIR/apps/coreir_examples/conv_bw
make all

cd Halide-to-Hardware/apps/hardware_benchmarks/tests/conv_3_3
make all
```
