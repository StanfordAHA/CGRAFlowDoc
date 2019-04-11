# Writing Applications
Below are some tips for writing Halide applications and some decisions
for scheduling to hardware.

## Algorithm
The algorithm defines the output pixels that are desired in the Halide
application. All basic C++ operators are overloaded to apply to Halide
functions. The supported operators are included in [Supported Operators](operations.md).

## Indexing
Image processing applications typically define pixels based on the local pixels
from a previous function. Therefore, values for an indexed function are defined
based on the `x` and `y` coordinate of other functions. This example performs
hot pixel suppression on Bayer images, where color channels are two pixels away
in all directions.
```C++
Expr max_value = max(max(input(x-2, y), input(x+2, y)),
                 max(input(x, y-2), input(x, y+2)));
Expr min_value = min(min(input(x-2, y), input(x+2, y)),
                 min(input(x, y-2), input(x, y+2)));

Func denoised("denoised");
denoised(x, y) = clamp(input(x, y), min_value, max_value);
```

## Reduction Domain
A common pattern used in image processing is convolution of an image with filter
weights. This can be done using a reduction domain. A single line of code can
define a sum of products:
```C++
Func input, kernel, convolution;

// Define x and y loops from -1 of length 3
RDom win(-1, 3, -1, 3);

kernel(x, y) = x + y;
convolution(x, y) = 0;  // set initial value of 0
convolution(x, y) += input(x + win.x, y + win.y) *
                     kernel(win.x, win.y);
```

The default schedule for a reduction domain is to have two loops, but in
hardware, this would create two counters. We can unroll these loops to
duplicate the multipliers and adders, and remove the counters.
```C++
convolution.update(0)
           .unroll(win.x).unroll(win.y);
```

## Select
Conditional statements do not translate well into hardware. Therefore, it is
recommended to use a version of a ternary or switch operator. In Halide, this
is provided using the `select` statement. This generates a mux in hardware. Below
shows how a `select` statement can be used for non-maximal suppression.
```C++
Expr is_max = in(x, y) > in(x-1, y-1) && in(x, y) > in(x, y-1) &&
    in(x, y) > in(x+1, y-1) && in(x, y) > in(x-1, y) &&
    in(x, y) > in(x+1, y) && in(x, y) > in(x-1, y+1) &&
    in(x, y) > in(x, y+1) && in(x, y) > in(x+1, y+1);
hw_output(x, y) = select( is_max, in(x, y), 0);
```

## Array of Funcs
Code can become a bit tedious due to writing all of the indices as above. 
Normally one would replace these with RDoms, but when intermediate values
are needed or the reduction operator is not defined (such as max), instead
an array of `Func`s can be used. One
can preserve the DAG structure while making it more readable using an array of 
`Func`s.
```C++
Func segment;
Func largest_value[16];
for (int i=0; i<16; ++i) {
  if (i==0) {
    largest_value[i](x,y) = segment(x,y,0);
  } else {
    largest_value[i](x,y) = max(segment(x,y,i), largest_value[i-1](x,y));
  }
}
```

## Linebuffers
A linebuffer is a memory element that has the minimum amount of size to
store the working set during execution. For example, for 3x3 convolution,
two rows and 3 pixels are needed from the input image during execution.
The Halide compiler will insert a linebuffer when specified in the schedule
with a linebuffer size to match the stencil size of the kernel.

## FIFOs
When there is convergence of `Func`s in the application, it is necessary that 
the hardware is constructed with delays through each parts match. When a DAG
diverges into separate `Func`s, a FIFO is needed when the delays are mismatched
from different linebuffer sizes. Any difference in linebuffer sizes should
be accompanied by a FIFO for the input that arrives early, so pixels arrive
at the same time.
