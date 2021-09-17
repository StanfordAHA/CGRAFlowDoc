# Writing Schedules
Below are some tips for writing Halide schedules.

## Schedule
While the algorithm defines the output pixels, the schedule determines how
to run on the specified hardware. These decisions can be used to improve the
runtime, code size, or utilization of the end application.

## Schedule Structure
The schedule typically follows a certain unique structure where the output is
scheduled first, and then it proceeds "backwards" toward the input. This is done
because some of the scheduling primitives refer to their output, such as `compute_at`.

Another quality to note is that the Halide scheduling language is not performed
sequentially, meaning that all of the scheduling calls will all occur for the program,
and the order of the calls does not matter too much.

Furthermore, while the Halide scheduling language is itself following syntax unique to
its own language, it is still embedded in C++. This means that you can still use printf
and `if` statements to control how the schedule is performed.

## Targets
After the application algorithm, one should specify the schedule for different
hardware targets. Typically, hardware targets require their own unique schedules
due to the different metrics and contraints on the hardware. For example, a CPU
and a custom CGRA accelerator have much different memory hierarchy and size, so the
corresponding tiling will have different sizes.

Each of these schedules usually exist in an if/else if/else block:
```C++
  if (get_target().has_feature(Target::Clockwork)) {  ...
```

## Clockwork Target Schedules
The Clockwork target schedule is the one used to target the CGRA. The memory file is
sent to Clockwork, while the compute is codegen'ed into a json file. Some of the
scheduling primitives are new for hardware, as well as some primitives (such as unroll)
have a different meaning for hardware targets.

### Bounding Image Sizes
The image sizes should be bounded since the hardware needs to have a size at compile-time.
This allows the memories to have a determined size. To bound the output width and height, use
`bound` like:
```C++
      const int outputWidth  = 64;
      const int outputHeight = 128;
      output.bound(x, 0, outputWidth);
      output.bound(y, 0, outputHeight);
```

Sometimes, input sizes also must be bound if they cannot be determined from the output size.
For example, the number of input channels for a DNN layer does not depend on the output. In
this case, an input bound is needed.

### Defining Hardware Boundaries
The hardware accelerator is part of an application that runs on the host processor. Scheduling
primitives are used to define the transfer to the accelerator and from the accelerator. These
primitives are `stream_to_accelerator` and `hw_accelerate`. `stream_to_accelerator` is used
on each input. It takes no arguments, and it is simply checked in the compiler to ensure that
the correct accelerator input is defined when creating the accelerator.

The output is defined by `hw_accelerate(compute_var, store_var)`. The function that has this
scheduling function is the output from the accelerator. The first argument is unused, but
defines what variable is a single cycle on the hardware. The second variable, `store_var` is
the first loop that is performed outside the accelerator. For example:
```C++
       hw_output
         .tile(x, y, xo, yo, xi, yi, 64, 128)
         .hw_accelerate(xi, xo);
       hw_input.
         .accelerator_input();
```

In this example, the loops `xi` and `yi` are size 64 and 128 respectively, and are executed
on the CGRA. The outer loops, `xo` and `yo` are performed outside the accelerator. This
effectively means that the accelerator is called `xo * yo` times to calculate the full output.

### Splitting, Reordering, and Tiling
There are scheduling primitives to separate loops and determine the loop order. `split` is used
to take a single loop variable and construct two nested loops from it. For example:
```C++
      output_glb
         .split(w, w_glb, w_cgra, 8)
```
In this example, the `w` loop is decomposed into a `w_cgra` loop of size 8 and a `w_glb` loop
that covers the rest of the loop iteration. Every 8 iterations of `w_cgra` increments `w_glb`
once.

To determine the order of your loop variables (including split variables), we can use `reorder`.
This lists the loops from inner-loop-variable to outermost-loop-variable. For example:
```C++
       output_glb
         .tile(x, y, x_glb,y_glb, x_cgra,y_cgra, tilesize_x,tilesize_y)
         .split(w, w_glb, w_cgra, k_oc)
         // reorder from inner to outermost
         .reorder(w_cgra, x_cgra, y_cgra,
                  w_glb, x_glb, y_glb);
```
We first split loops using `tile` and `split`. Afterwards, we have 6 separate loop variables
for the function `output_glb`. The `reorder` determines the loop ordering of these 6 variables.

Finally, there is `tile`, which is just syntatic sugar for `split` and `reorder`. This scheduling
primitive was created since splitting and reordering images into tiles usually is a common process.
Two equivalent schedules would thus be:
```C++
       output_glb
         .tile(x, y, x_glb,y_glb, x_cgra,y_cgra, tilesize_x,tilesize_y);
```
and:
```C++
       output_glb
         .split(x, x_glb, x_cgra, tilesize_x)
         .split(y, y_glb, y_cgra, tilesize_y)
         .reorder(x_cgra, y_cgra, x_glb, y_glb);
```

### Duplicating Compute
Duplicating compute is useful to perform a computation in fewer cycles. This is done by performing
multiple iterations of a loop in a single cycle. In hardware, this is done by duplicating compute
resources. Similar to HLS semanatics, we use `unroll` to express this hardware duplication.

One of the common uses of hardware duplication is for unrolling a reduction domain to perform in
a single cycle.
```C++
  RDom win(0, blockSize, 0, blockSize);
  blur(x, y) += kernel(win.x, win.y) * hw_input(x+win.x, y+win.y);
  
   blur.update()
      .unroll(win.x)
      .unroll(win.y);
```
This example shows that a reduction domain `win` is used to perform a convolution. This computation
would normally be performed in `blockSize * blockSize` iterations. We can get a handle on the
adder chain by using `blur.update()`, and then unroll this computation using `unroll` to duplicate
the compute completely to perform it in a single cycle.

We can further unroll computation to perform multiple pixels per cycle. We typically want to unroll
the innermost dimension to allow for the memories to input the subsequent values more easily.
For example:
```C++
   blur.update()
      .unroll(win.x)
      .unroll(win.y);
      .unroll(x, 3, TailStrategy::RoundUp);
```
This schedule has unrolled the reduction domain to perform in a single cycle, and thens the
innermost loop, `x`, is unrolled by 3 to make it 3 pixels/cycle. The `TailStrategy::RoundUp` is
used to increase the loop size in case the image width (the extent of `x`)  is not divisible by 3.

Note that even after unrolling the compute, the rest of the application should be equally unrolled.
Duplicating hardware at one compute kernel is usually not helpful if the rate of the other kernels
are not similarly duplicated. Furthermore, even if all of the compute kernels are unrolled, the
initializations and memory transfers also need to also support the same rate. These schedules will
thus commonly also include lines like below:
```C++
   // unroll the initialization
   blur.unroll(x, 3, TailStrategy::RoundUp);

   // unroll the output and input memories
   hw_output.unroll(xi, 3, TailStrategy::RoundUp);
   hw_input.unroll(x, 3, TailStrategy::RoundUp);
```

### Creating Memories
In addition to compute, it is important to schedule the memories properly. Without any scheduling
all compute is by default computed in-line. This means that no reuse is used and lots of compute
resources are created. For most applications, it is worth to create memories anywhere you can,
because the memory resources are usually abundant and the reduction in compute resources is
significant.

A memory can be created for any Func, but it is only useful (and becomes a memory) when there
is some sort of reuse. This can be seen in the Halide algorithm when there is an offset in
the index, or a reduction domain is used. For example:
```C++
   blur_y(x, y) = (blur_x(x, y) + blur_x(x, y+1) + blur_x(x, y+2))/3;

   blur(x, y) += kernel(win.x, win.y) * hw_input(x+win.x, y+win.y);
```

For these two, `blur_x`, `kernel`, and `hw_input` should be stored in a memory. To schedule these
Funcs in memory tiles, use `compute_at` and `store_at`:
```C++
   blur_x.compute_at(hw_output, xo).store_at(hw_output, xo);
   
   hw_input.compute_at(hw_output, xo).store_at(hw_output, xo);
```
The arguments are used to specify which Func and loop variable where this memory should be created.
Effectively, a memory is created at the specified loop level, and any reuse within that will be
captured. For a hardware accelerator without hierarchy, this is commonly the output Func (that is
accelerated) and the store (second) variable. Note that due to the requirements of a `store_at` and
`compute_at`, usually only `compute_at` is necessary to specify, since it will automatically store
at the same level.

As a further clarification of this process, it may seem odd that the compute level is the same as
the store level. Usually in Halide this means that the computation would be computed first at the
specified level, and only afterwards continue the computation. This would typically lead to a much
larger memory storage at each level than necessary, as well as create serial code. Instead, Clockwork
prefers this specification, but will instead create the line buffers that are expected. In effect,
all memories therefore are scheduled this way, and Clockwork fixes the compute level based on its
own analysis.

### More Hardware Boundaries
Although `hw_accelerate` and `stream_to_accelerator` are the basic ways of specifying the boundary,
there are more complications internally. In `src/Func.cpp`, one will find that `stream_to_accelerator()`
is simply syntatic sugar for `.in()` as well as `is_accelerator_input`. In effect, this call is
creating the necessary memory hierarchy needed for Clockwork. To have more control of these input and
output labels, one can find `accelerator_input()`, `accelerator_output()`, and `accelerate_call_output()`.
Not all of these scheduling primitives are fully fleshed out, but they can be used to specify more
advanced application structures such as create memory hierarchies and use multiple `hw_accelerate`
calls in a single application.

### Creating Memory Hierarchy
Creating a memory hierarchy is a matter of specifying a series of copies from larger memories
to smaller memories. In effect, the computation between them are NOPs, and then tiling should
occur to specify the differing sizes.

Halide provides the means of performing these copies using `.in()`. The generated Func will end with
`global_wrapper`, but no computation will occur between them. When creating the memory hierarchy,
one should create a hierarchy at both the input and output, and align the copies using `compute_at`.

An example of a memory hierarchy using three levels:
```C++
      hw_output.in().compute_root();

      hw_output.in()
        .tile(x, y, xo, yo, xi, yi, glbWidth, glbHeight)
        .hw_accelerate(xi, xo);
      hw_output.in().store_in(MemoryType::GLB);

      Var xii, yii, xio, yio;
      hw_output
        .tile(x, y, xio, yio, xii, yii, tileWidth, tileHeight);
      hw_output.compute_at(hw_output.in(), xo);

      blur.compute_at(hw_output, xio);
      
      blur_unnormalized.update()
        .unroll(win.x, blockSize)
        .unroll(win.y, blockSize);
      blur_unnormalized.compute_at(hw_output, xio);

      hw_input.in().in().compute_at(hw_output, xio); // represents the mem tile

      hw_input.in().compute_at(hw_output.in(), xo);
      hw_input.in().store_in(MemoryType::GLB);
      
      hw_input.compute_root()
        .accelerator_input();
```
Here you'll notice that the global buffer level at the input and output are labelled using
`store_in(MemoryType::GLB)`. The accelerator output is the GLB `hw_output.in()` meaning that the
input GLB, `hw_input.in()` is computed at this same level: `hw_input.in().compute_at(hw_output.in(), xo)`.
The inner compute on the CGRA has its own tiling, `hw_output.tile()`, as well as its own input
memory tile that is computed at the CGRA output `hw_input.in().in().compute_at(hw_output, xo)`.
This effectively creates a three level memory hierarchy with the levels: host, GLB, and memory tile.

## CPU Target Schedules
CPU schedules are different from Clockwork and hardware targets. The memory sizes are different,
and scheduling primitives such as `parallel` and `vectorize` play a key role in improving runtimes.
Furthermore, `compute_at` and `store_at` play a far more complex and important role to ensure that
values fit in different cache sizes. There are many tradeoffs to consider when trying to achieve
the fastest CPU implementation for any given machine.

## Debugging Schedules
There are several places to look to identify whether the schedule is performing how you expect. By
modifying `src/Lower.cpp`, one can output the HalideIR at any stage (I recommend especially the final
stage before codegen). In addition, the generated code (`bin/<app>_memory.cpp` and `bin/<app>_compute.h`)
can help identify what calls are being created in the hardware.

An example generated loopnest is below:
```C++
if (!(_halide_buffer_is_bounds_query(input.buffer) || _halide_buffer_is_bounds_query(output.buffer))) {
  assert((_halide_buffer_get_type(input.buffer) == (uint32)67585), halide_error_bad_type("Input buffer input", _halide_buffer_get_type(input.buffer), (uint32)67585))
  ...
  assert((_halide_buffer_get_host(output.buffer) != reinterpret((void *), (uint64)0)), halide_error_host_is_null("Output buffer output"))
  realize hw_input.stencil([0, 64], [0, 64]) {
    produce hw_input.stencil {
      for (hw_input.s0.y, 0, 64) {
        for (hw_input.s0.x, 0, 64) {
          hw_input.stencil(hw_input.s0.x, hw_input.s0.y) = uint16(input[(((_halide_buffer_get_stride(input.buffer, 1)*hw_input.s0.y) - (_halide_buffer_get_min(input.buffer, 0) + (_halide_buffer_get_min(input.buffer, 1)*_halide_buffer_get_stride(input.buffer, 1)))) + hw_input.s0.x)])
        }
      }
    }
    consume hw_input.stencil {
      realize hw_output_global_wrapper.glb.stencil([0, 62], [0, 62]) {
        produce hw_output_global_wrapper.glb.stencil {
          produce _hls_accelerator.hw_output_global_wrapper {
            produce _hls_target.hw_output_global_wrapper {
// This represents the input global buffer
              realize hw_input_global_wrapper.glb.stencil([0, 64], [0, 64]) {
                produce hw_input_global_wrapper.glb.stencil {
                  for (hw_input_global_wrapper.s0.y, 0, 64) {
                    for (hw_input_global_wrapper.s0.x.x, 0, 64) {
                      hw_input_global_wrapper.glb.stencil(hw_input_global_wrapper.s0.x.x, hw_input_global_wrapper.s0.y) = hw_input.stencil(hw_input_global_wrapper.s0.x.x, hw_input_global_wrapper.s0.y)
                    }
                  }
                }
// This section is for the CGRA MEMs and PEs
                consume hw_input_global_wrapper.glb.stencil {
                  realize hw_output.stencil([0, 62], [0, 62]) {
                    produce hw_output.stencil {
                      realize hw_input_global_wrapper_global_wrapper.stencil([0, 64], [0, 64]) {
                        produce hw_input_global_wrapper_global_wrapper.stencil {
                          for (hw_input_global_wrapper_global_wrapper.s0.y, 0, 64) {
                            for (hw_input_global_wrapper_global_wrapper.s0.x.x, 0, 64) {
                              hw_input_global_wrapper_global_wrapper.stencil(hw_input_global_wrapper_global_wrapper.s0.x.x, hw_input_global_wrapper_global_wrapper.s0.y) = hw_input_global_wrapper.glb.stencil(hw_input_global_wrapper_global_wrapper.s0.x.x, hw_input_global_wrapper_global_wrapper.s0.y)
                            }
                          }
                        }
                        consume hw_input_global_wrapper_global_wrapper.stencil {
                          realize blur_unnormalized.stencil([0, 62], [0, 62]) {
                            produce blur_unnormalized.stencil {
                              for (blur_unnormalized.s0.y, 0, 62) {
                                for (blur_unnormalized.s0.x.x, 0, 62) {
                                  blur_unnormalized.stencil(blur_unnormalized.s0.x.x, blur_unnormalized.s0.y) = (uint16)0
                                }
                              }
                              for (blur_unnormalized.s1.y, 0, 62) {
                                for (blur_unnormalized.s1.x.x, 0, 62) {
                                  blur_unnormalized.stencil(blur_unnormalized.s1.x.x, blur_unnormalized.s1.y) = ((hw_input_global_wrapper_global_wrapper.stencil(blur_unnormalized.s1.x.x, blur_unnormalized.s1.y)*(uint16)24) + (blur_unnormalized.stencil(blur_unnormalized.s1.x.x, blur_unnormalized.s1.y) + ((hw_input_global_wrapper_global_wrapper.stencil((blur_unnormalized.s1.x.x + 1), blur_unnormalized.s1.y)*(uint16)30) + ((hw_input_global_wrapper_global_wrapper.stencil((blur_unnormalized.s1.x.x + 2), blur_unnormalized.s1.y)*(uint16)24) + ((hw_input_global_wrapper_global_wrapper.stencil(blur_unnormalized.s1.x.x, (blur_unnormalized.s1.y + 1))*(uint16)30) + ((hw_input_global_wrapper_global_wrapper.stencil((blur_unnormalized.s1.x.x + 1), (blur_unnormalized.s1.y + 1))*(uint16)37) + ((hw_input_global_wrapper_global_wrapper.stencil((blur_unnormalized.s1.x.x + 2), (blur_unnormalized.s1.y + 1))*(uint16)30) + ((hw_input_global_wrapper_global_wrapper.stencil(blur_unnormalized.s1.x.x, (blur_unnormalized.s1.y + 2))*(uint16)24) + ((hw_input_global_wrapper_global_wrapper.stencil((blur_unnormalized.s1.x.x + 2), (blur_unnormalized.s1.y + 2))*(uint16)24) + (hw_input_global_wrapper_global_wrapper.stencil((blur_unnormalized.s1.x.x + 1), (blur_unnormalized.s1.y + 2))*(uint16)30))))))))))
                                }
                              }
                            }
                            consume blur_unnormalized.stencil {
                              realize blur.stencil([0, 62], [0, 62]) {
                                produce blur.stencil {
                                  for (blur.s0.y, 0, 62) {
                                    for (blur.s0.x.x, 0, 62) {
                                      blur.stencil(blur.s0.x.x, blur.s0.y) = (blur_unnormalized.stencil(blur.s0.x.x, blur.s0.y)/(uint16)256)
                                    }
                                  }
                                }
                                consume blur.stencil {
                                  for (hw_output.s0.y.yii, 0, 62) {
                                    for (hw_output.s0.x.xii.xii, 0, 62) {
                                      hw_output.stencil(hw_output.s0.x.xii.xii, hw_output.s0.y.yii) = blur.stencil(hw_output.s0.x.xii.xii, hw_output.s0.y.yii)
                                    }
                                  }
                                }
                              }
                            }
                          }
                        }
                      }
                    }
// This represents the output global buffer
                    consume hw_output.stencil {
                      for (hw_output_global_wrapper.s0.y.yi, 0, 62) {
                        for (hw_output_global_wrapper.s0.x.xi.xi, 0, 62) {
                          hw_output_global_wrapper.glb.stencil(hw_output_global_wrapper.s0.x.xi.xi, hw_output_global_wrapper.s0.y.yi) = hw_output.stencil(hw_output_global_wrapper.s0.x.xi.xi, hw_output_global_wrapper.s0.y.yi)
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        }
        consume hw_output_global_wrapper.glb.stencil {
          assert(((0 <= _halide_buffer_get_min(output.buffer, 1)) && ((_halide_buffer_get_extent(output.buffer, 1) + _halide_buffer_get_min(output.buffer, 1)) <= 62)), halide_error_explicit_bounds_too_small("y", "output", 0, 61, _halide_buffer_get_min(output.buffer, 1), ((_halide_buffer_get_extent(output.buffer, 1) + _halide_buffer_get_min(output.buffer, 1)) + -1)))
          assert(((0 <= _halide_buffer_get_min(output.buffer, 0)) && ((_halide_buffer_get_extent(output.buffer, 0) + _halide_buffer_get_min(output.buffer, 0)) <= 62)), halide_error_explicit_bounds_too_small("x", "output", 0, 61, _halide_buffer_get_min(output.buffer, 0), ((_halide_buffer_get_extent(output.buffer, 0) + _halide_buffer_get_min(output.buffer, 0)) + -1)))
          produce output {
            for (output.s0.y, 0, 62) {
              for (output.s0.x, 0, 62) {
                output[((_halide_buffer_get_stride(output.buffer, 1)*output.s0.y) + output.s0.x)] = uint8(hw_output_global_wrapper.glb.stencil(output.s0.x, output.s0.y))
              }
            }
          }
        }
      }
    }
  }
}
```

Above is a Gaussian blur with a memory hierarchy targetted for a CGRA. The sections are marked based on 
intention. The loopnest can be analyzed for code intedend for the CGRA with memories as well as global buffer input
and output. The above is the default printout for  HalideIR in a form that is close to the C++ implementation.

## Extending the Scheduling Language
While most of the functionality for the scheduling language exists in Halide, one might find that
extending the scheduling to be needed. This might be for additional functionality or additional
syntatic sugar. These modifications will start in `src/Func.cpp`.
