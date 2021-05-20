This repo is obsolete, it has most of the applications that the CGRAFlow is testing. This
old repo has been phased out.

* Old repo:
  https://github.com/jeffsetter/Halide_CoreIR

You should be able to compile an application by invoking `make` command
in the top level folder

```
cd Halide_CoreIR/apps/coreir_examples/conv_bw
make all
```

# Usage
## Applications
The applications can be found for the Halide_CoreIR repo in `apps/coreir_examples` and `apps/coreir_tests`.

## Make targets
For the Halide_CoreIR repo, generation has three basic targets: `make clean`, 
`make design_top.json`, and `make out.png`. This will first remove old generated
files, then create the CoreIR file, and last test the CoreIR.
```make
make clean             # remove generated files
     design_top.json   # create CoreIR
     graph.png         # use graphviz to create visualization of circuit 
     out.png           # create output file
     test              # create CoreIR and check if circuit matches cached result
     update_golden     # update reference CoreIR json file
```
## Directory for Halide_CoreIR repository
```
Halide_CoreIR
└── apps
    ├── coreir_tests                       // contains simpler test cases
    └── coreir_examples                    // contains all apps compiled to coreir
        └── conv_bw                        // one of the apps: does 3x3 convolution
            ├── Makefile                   // specifies commands for 'make'
            ├── pipeline.cpp               // contains Halide algorithm and schedule
            ├── design_top_golden.json     // reference coreir output expected by compiler
            ├── input.png                  // input image for testing
            ├── run.cpp                    // runs input image with design to create ouput image
            │
            │                              //// Running 'make all' generates:
            ├── design_top.json            // generated CoreIR design
            ├── design_top.txt             // generated graphiz representation of CoreIR
            ├── graph.png                  // generated graphiz image of circuit (using 'make graph.png')
            └── out.png                    // output image created during testing
```

# Testing
To ensure that the output CoreIR circuits are correct, one is encouraged
to test the circuit execution. Halide can be used to generate CPU code, 
which can be used as a golden model. Input images are used for the CPU
and CoreIR models, and then the output images are then compared pixel by
pixel. Each test case has uses `run.cpp` to run each model and compare
each pixel.

## CoreIR interpreter
To test CoreIR, pixels are set for the input pixel by pixel. For each pixel,
the CoreIR is executed, and an output is retrieved on each cycle. A sample
of the body of CoreIR execution from `run.cpp` is shown below:
```C++
for (int y = 0; y < input.height(); y++) {
  for (int x = 0; x < input.width(); x++) {
    for (int c = 0; c < input.channels(); c++) {
      // set input value
      state.setValue("self.in_arg_1_0_0", BitVector(16, input(x,y,c)));
      // propogate to all wires
      state.exeCombinational();
          
      // read output wire
      out_coreir(x,y,c) = state.getBitVec("self.out_0_0").to_type<uint16_t>();
      if (x>=stencil_size && y>=stencil_size && out_native(x-stencil_size, y-stencil_size, c) != out_coreir(x, y, c)) {
        printf("out_native(%d, %d, %d) = %d, but out_coreir(%d, %d, %d) = %d\n",
               x-stencil_size, y-stencil_size, c, out_native(x-stencil_size, y-stencil_size, c),
               x, y, c, out_coreir(x, y, c));
        success = false;
      }

      // give another rising edge (execute seq)
      state.exeSequential();

    }
  }
}

```

# Hardware Lowering
There are a pair of lowering passes that are unique to hardware. The current
implementation uses two custom passes to achieve this; however, in the 
future some of the more recent analyses might be adopted to perform these
passes in a more robust fashion.

## Linebuffer Extraction
In this pass, the parameters for the linebuffers are lifted from the usage
of the functions. This analysis must:
- identify where the user asked for a linebuffer
- image size (what is the width of the input image)
- output stencil size (i.e., width and height of the convolution kernel)
- input pixel rate (how many pixels are inputted each cycle)

The location of a linebuffer is found by matching the loop variables
annotated by the `accelerate` scheduling primitive when defining the 
hardware accelerator. By matching the `store_at` and `compute_at` variables,
we know that the storage can be optimally created as a linebuffer.

The image size is identified by Halide's bounded-size analysis pass. This
pass traces from the output to the desired function to find what footprint
is needed for the function. This defines the `Func`'s minimum and maximum
`x` and `y` index. The maximum width is then used to define the linebuffer
row's width.

The output stencil size is needed to define how many values are needed on
each iteration. These values are typically a rectangular region of the image,
and in hardware must be available on the cycle. This parameter is used to
control how many registers are created between each rowbuffer. Since a
rectangular region is expected, the box-touched analysis pass can be used
to find which pixel indices are used in each cycle. This box should match
the reduction domain's size for a convolution.

Finally, the input pixel rate defines how many pixels are inserted into
the linebuffer on each cycle. For every cycle iteration, there are a constant
number of pixels that are produced. Based on this constant rate, there is
some overlap of the pixels used from previous iterations, but also some new
pixels. This pass finds how many new pixels are needed by comparing the
difference between iteration `x` and iteration `x+1`. The number of pixels
between these two iterations is expected to be a constant; if not the
compiler throws an error, since a linebuffer cannot satisfy a varying
number of input pixels on different cycles.

## Stream Optimization
With the linebuffers discovered from the previous analysis, the HalideIR 
must be modified to use these new memories. Arrays are replaced, and indexing
is replaced with new indices into the linebuffer output stencil.

In addition, the Halide compiler usually produces sequential code for CPU execution.
However, for hardware, all code segments at the iteration variable execute
simultaneously. A stream of input pixels and stream of output pixels are
expected enter and exit the accelerator. The HalideIR is modified to match
how hardware behaves with streaming operators.