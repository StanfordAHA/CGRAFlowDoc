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

## Testing locally
A Makefile is used to create the designs and test them. After modifing the
compiler and/or an application, the following line will rebuild the compiler
and test the application.
```
cd apps/coreir_examples/${APP}
make -j compiler && make test
```

If the compiler has been modified between runs, all test cases should be 
tested. To test all applications, run the commands below.
This will compile, execute, and compare the images for all of the test cases
in parallel using all threads in one's machine. Upon completion, the
result of all test cases are outputted to the terminal screen.
```
cd apps/coreir_examples
make -j compiler && make -j testall
```

## Travis CI
Travis CI runs on each commit. It will start from a fresh VM and
compile Halide from scratch. Following this, it will run a subset of the
tests and applications. Due to the runtime limit of Travis, not all applications
are tested.