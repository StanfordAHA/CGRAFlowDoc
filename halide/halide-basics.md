# Halide Basics

## Language
Halide is an image-processing DSL that makes it easy to design algorithms and
search for a schedule to make it run fast. _Algorithms_ are defined by pure
functions that define arrays computed based on indices (x and y cooordinate) where 
the loops are implicit. The _schedule_ then allows the user to trade off locality, 
recomputation, and parallelism using different loop transformations.

## Limitations
Halide itself is not Turing complete (due to the limited nature of loops).
To create hardware, one must also limit designs to feed-forward pipelines and
limit recursion to a bounded depth. However, these limitations are not
commonly major issues in the image-processing domain, and when creating
synthesiable hardware.

## Example: 3x3 convolution using separable filter
Below is an example Halide program. Note the absence of loops since they
are implicit and loop bounds are defined during execution.
```C++
class MyPipeline() {
  ImageParam input;
  Func hw_input, blur_x, blur_y, output;
  Var x, y, xi, yi;
  std::vector<Arg> args;

  MyPipeline() {
    // The algorithm - no storage or order
    hw_input(x, y) = input(x, y);
    blur_x(x, y) = (hw_input(x-1, y) + hw_input(x, y) + hw_input(x+1, y))/3;

    blur_y(x, y) = (blur_x(x, y-1) + blur_x(x, y) + blur_x(x, y+1))/3;
    output(x, y) = blur_y(x, y);

    args.push_back(input);
  
    // The schedule - defines order, locality; implies storage
    blur_y.tile(x, y, xi, yi, 64-2, 64-2)
          .compute_root()
          .accelerate({hw_input}, xi, xo, {});
          
    blur_x.linebuffer();
  }

  // Compile coreir to file
  void compile_coreir() {
    Target coreir_target = get_target_from_environment();
    output.compile_to_coreir("pipeline_coreir.cpp", args, 
                             "pipeline_coreir", coreir_target);
  }

}
```

## Hardware Targets
Halide is defined to create programs that can be run on a variety of targets,
including CPU and GPU. With the ability to produce code for CPU, designs can
run on a CPU to create reference images to verify execution.

Additionally, this work adds CoreIR for a hardware-specific target intended
to create verilog or mapping to our CGRA. Note that scheduling is interpretted
slightly different for hardware, and different tradeoffs exist. This is explored
in more depth at [Writing Applications](writing-apps.md).



