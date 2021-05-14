# Usage

## Installation
See [Installation](installation.md) section for how to install
dependencies and how to compile Halide.

## Applications
The applications for the Halide-to-Hardware repo exist in `apps/hardware_examples/apps`
and `apps/hardware_examples/tests` where the distinction is that tests tend to focus
on a much narrower scope as compared to apps.

For the Halide-to-Hardware repo, the targets are build into a series of small steps.
The basic steps are: `make clean`, `make clockwork`, and `make bin/output_clockwork.png`.
Some targets are duplicated with new target names. The definition
of all of these targets can be found in `apps/hardware_benchmarks/hw_support/hardware_targets.mk`.
```make
make clean                     # remove generated files (bin directory)
     compiler                  # compile updates to Halide compiler
     generator                 # create Halide generator
     design-clockwork          # create Clockwork design
     design-cpu                # create CPU design
     bin/output_cpu.png        # create output file using CPU implementation
     run-cpu                   # create output file using CPU implementation
     bin/output_clockwork.png  # create output file using Clockwork implementation
     run-clockwork             # create output file using Clockwork implementation
     compare-clockwork-cpu     # compare Clockwork output file to CPU output image
     golden                    # update reference CoreIR json file and output image
     mem                       # run the Clockwork compiler and create design_top.json
     memtest                   # test the full Clockwork/CoreIR application using verilator
```

## Application Folder
The following is a list of the files that are important for generating
images and are used further in the flow. The CoreIR json files are created
for mapping to the CGRA. Also, an output image, out.png, is generated as a 
the reference image to determine if execution is correct. `out.png` is 
created using a CPU implementation of Halide, and its output should be
used for validation during CoreIR interpretation and CGRA simulation.

### Directory for Halide-to-Hardware repository
```
Halide_CoreIR
└── apps
    └── hardware_benchmarks                      // contains simpler test cases
        ├── apps                                 // contains all apps compiled to Clockwork
        └── tests                                // contains all simpler test cases
            └── conv_3_3                         // one of the tests: does 3x3 convolution
                ├── Makefile                     // specifies commands for 'make'
                ├── conv_3_3_generator.cpp       // contains Halide algorithm and schedule
                ├── input.png                    // input image for testing
                ├── process.cpp                  // runs input image with design to create ouput image
                ├── golden                       // this holds all expected output files
                │   └── golden_output.png        // output image expected
                │                                
                └── bin                          //// Running 'make all' generates in this folder:
                    ├── design_top.json          // generated CoreIR design after Clockwork mapping
                    ├── conv_3_3_memory.cpp      // generated Clockwork memory file
                    ├── conv_3_3_compute.json    // generated CoreIR compute kernels
                    ├── clockwork_testscript.cpp // generated script used to run the Clockwork files
                    ├── output_cpu.png           // output image created using CPU implementation
                    └── output_clockwork.png     // output image created during testing; should be equivalent to output_cpu.png
```

## Running Multiple Applications

In order to run multiple tests, the Makefile in the tests or apps directory is used. There
are a couple of make targets specifically to run on all tests, such as:

```
  make list      # list all of the tests, and which tests are not classified into a test suite
  make check     # check each test for generated coreir, generated ubuffers, correct output
  make cleanall  # clean bin folders for each test
  make suites    # list all of the suites that exist
```

The make targets in these directories run their associated target in individual tests.
For example, to generate the coreir for conv_3_3, one can run:

```sh
  cd apps/hardware_targets/tests
  make conv_3_3-clockwork
```

This results in conv_3_3 creating the Clockwork files, which then creates the
Clockwork output file, `tests/conv_3_3/bin/conv_3_3_memory.cpp`. Note that the
output is silent besides a FAILED/PASSED. To run with debug output, change
to the desired directory and run `make clockwork` there.

Test targets are:
```sh
  make conv_3_3-clockwork # create Clockwork hardware design (conv_3_3_memory.cpp)
               -run-cpu   # create output image using the CPU (output_cpu.png)
               -compare   # compare Clockwork output image and CPU output image. If matching, save as output.png
               -check     # check for generated Clockwork, CPU, and correct output images
               -clean     # delete bin folder for the test
```

Besides individual tests, one can run a command on multiple tests. The
tests are grouped into different test suites. The test suites are:
```
  all     # all tests in a sorted order
  ops     # single operators, such as + < &&
  fpops   # single operators using floating point, such as f+ f< sin exp
  conv    # convolutions of different sizes
  fpconv  # convolutions of different sizes using floating point numbers
  inout   # tests of the interface, such as two inputs, bit input, floating point output
  mem     # tests of the memory tile, such as ROM, downsample, accumulation
```

For example, to run a comparison to check Clockwork correctness on the convolutions, use;
```
  make conv-compare
```

Note that parallelism works with these targets, so to run the convolutions in parallel:
```
  make conv-compare -j8
```
A list of all of the Makefile targets can be found in `apps/hardware_benchmarks/include.mk`.
