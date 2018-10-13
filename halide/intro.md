# Introduction
## Overview
(keyi: copied from http://halide-lang.org/)

Halide is a programming language designed to make it easier to write
high-performance image and array processing code on modern machines.
Halide currently targets:

* CPU architectures: X86, ARM, MIPS, Hexagon, PowerPC
* Operating systems: Linux, Windows, macOS, Android, iOS, Qualcomm QuRT
* GPU Compute APIs: CUDA, OpenCL, OpenGL, OpenGL Compute Shaders, Apple Metal,
  Microsoft Direct X 12

Rather than being a standalone programming language, Halide is embedded in C++.
This means you write C++ code that builds an in-memory representation of a
Halide pipeline using Halide's C++ API. You can then compile this
representation to an object file, or JIT-compile it and run it in the same
process.

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

## Getting Started
There are two Halide repo that Jeff is maintaining. Although one of them is
obsolete, it has most of the applications that the CGRAFlow is testing. We
expect the old repo is phasing out soon.

* Old repo:
  https://github.com/jeffsetter/Halide_CoreIR
* New repo:
  https://github.com/jeffsetter/Halide-to-Hardware/tree/coreir_target

You should be able to compile both of them simply by invoking `make` command
in the top level folder.
